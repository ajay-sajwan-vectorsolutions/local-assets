# Business Logic — Vector Web Components Dashboard

## State Variables

All application state lives in plain JS variables in `index.html`.

| Variable | Type | Purpose |
|---|---|---|
| `allResults` | `Array` | Full scan output — one entry per component-per-file-per-repo found |
| `filteredResults` | `Array` | Active dataset after filters applied; used for pagination and detail expansion |
| `orgRepos` | `Array<string>` | All repo names fetched from the GitHub org |
| `selectedRepos` | `Set<string>` | Repos the user has chosen to scan |
| `reposLoaded` | `boolean` | Guards the repo picker trigger against opening before `loadOrgRepos` completes |
| `productMap` | `Object` | `{ repoName → "Product value" }` from org custom properties |
| `repoClassMap` | `Object` | `{ repoName → "Repo-Class value" }` from org custom properties |
| `npmCache` | `Object` | `{ packageName → { latest, versions } \| null }` — memoizes npm registry responses |
| `currentPage` | `number` | Current pagination page (1-based) |
| `pageSize` | `number` | Rows per page (25 / 50 / 100) |
| `scanAbort` | `AbortController \| null` | Created at scan start; `null` when no scan is running |
| `chartByRepo` | Chart.js instance | Destroyed and recreated on every `renderDashboard` call |
| `chartFreshness` | Chart.js instance | Destroyed and recreated on every `renderDashboard` call |

---

## GitHub API Layer

All GitHub API calls go through a single helper:

```js
async function ghFetch(url, token, signal)
```

- Adds `Authorization: token <PAT>` and `Accept: application/vnd.github.v3+json` headers.
- Returns parsed JSON on success.
- Returns `null` on 404 (silently — used to detect missing files/dirs).
- Throws for any other non-OK status.
- Passes the `signal` from `AbortController` through to `fetch()`.

`fetchFileContent(fullRepoPath, filePath, token, signal)` builds on `ghFetch` to read a file, base64-decode `data.content`, and JSON-parse it. Returns `null` on any failure.

---

## Custom Property Fetching

Custom org properties (`Product`, `Repo-Class`) are fetched at **Load Repos** time alongside the repo list.

### Primary — org-level bulk endpoint
```
GET /orgs/:org/properties/values?per_page=100&page=N
```
Paginated. Each item contains a `repository_name` and a `properties` array. The code extracts `Product` and `Repo-Class` values into `productMap` and `repoClassMap`.

Requires the `read:org` PAT scope. If this call fails or returns no data, `propsLoaded` stays `false` and a warning is shown in the status bar.

### Fallback — per-repo endpoint
```
GET /repos/:org/:repo/properties/values
```
Called for each scanned repo that has no entry in `productMap` yet. Silently skipped on failure.

### Filter population
After a scan completes, `populateResultFilters(results)` walks `allResults`, looks up each result's repo in `productMap` / `repoClassMap`, and builds the unique sorted option lists for the Product and Repo-Class `<select>` dropdowns.

---

## Scan Flow

### Phase 0 — GitHub Code Search pre-filter

Before scanning any repo, the dashboard uses GitHub's Code Search API to narrow the list:

```
GET /search/code?q="@vector-web-components" filename:package.json org:<org>&per_page=100&page=N
```

Results are paginated. Repo names from `item.repository.name` are collected into a `Set`. The user's selected repos are then intersected with this set — only repos that appear in the search results proceed to Phase 1.

**Fallback:** if Code Search fails (rate limit, permission error, etc.), `reposToScan` falls back to the full list of selected repos. The scan continues without pre-filtering.

Progress: 0 → 10%.

---

### Phase 1 — Concurrent repo scanning

Repos are scanned in **batches of 5** using `Promise.allSettled`. This keeps GitHub API pressure reasonable while parallelizing work.

```js
const CONCURRENCY = 5;
for (let i = 0; i < reposToScan.length; i += CONCURRENCY) {
  const batch = reposToScan.slice(i, i + CONCURRENCY);
  const batchResults = await Promise.allSettled(batch.map(repo => scanRepo(...)));
  // accumulate fulfilled results; collect errors from rejected
}
```

`AbortError` from any batch result is re-thrown immediately to stop the scan.

Progress: 10 → 40%.

#### `scanRepo(org, repo, token, signal)`

For a single repo, reads:
1. `package.json` at the repo root.
2. The contents of each of three monorepo directories: `packages/`, `apps/`, `libs/`. For each subdirectory found, reads `<subdir>/<name>/package.json`.

For each `package.json` found, `extractVectorComponents` is called for all three dependency types: `dependencies`, `devDependencies`, `peerDependencies`. It filters entries whose name starts with `@vector-web-components/` and returns `{ name, version, depType }` objects.

Each found component is pushed to `allResults` as:
```js
{ repo, file, name, version, depType }
```

---

### Phase 2 — Version resolution

For each unique package name found in `allResults`, the dashboard attempts to resolve the latest available version. Progress: 40 → 95%.

#### Strategy A — GitHub lib repo (preferred)

`resolveFromGitHubRepo(libRepoPath, token, signal)` runs four steps in order:

**Step 1 — Fetch GitHub releases**
```
GET /repos/:libRepo/releases?per_page=100&page=N
```
All releases are collected for use as changelog data in the detail panel.

**Step 2 — Fetch GitHub tags**
```
GET /repos/:libRepo/tags?per_page=100&page=N
```
Tag names are stored in `allTags` for timeline construction.

**Step 3 — Directory scan for package.json files**

Iterates through four candidate directories: `packages`, `components`, `libs`, `modules`, and the repo root (`""`). For each directory listing returned, reads `<dir>/<name>/package.json` for every subdirectory. If a package's `name` starts with `@vector-web-components/`, its `version` field is recorded in `latestVersions`. Stops scanning further directories once any components are found.

**Step 4 — Tag parsing (fallback for non-directory resolution)**

If the directory scan found no components, the code parses version numbers from tag names using three regex patterns:

| Pattern | Example tag | Extracted package |
|---|---|---|
| `@vector-web-components/<name>@<version>` | `@vector-web-components/button@2.1.0` | `@vector-web-components/button` |
| `<name>@<version>` (semver format) | `button@2.1.0` | `@vector-web-components/button` |
| `v<version>` (single-package repo) | `v2.1.0` | collected under `__root__` key |

For each component, the highest version (by semver comparison) across all matching tags is used as `latestVersions[name]`.

For `__root__` tags (single-package repo), the lib repo's root `package.json` is fetched to resolve the package's scoped name.

**Step 5 — Root package.json last resort**

If no versions were resolved by any of the above methods, reads the lib repo's root `package.json` directly and uses its `name` and `version`.

#### Strategy B — npm registry fallback

If Strategy A produced no `latestVersion` for a package, `fetchNpmLatest(packageName, signal)` queries:
```
GET https://registry.npmjs.org/<package-name>
```
Results are memoized in `npmCache` (keyed by package name). Extracts `dist-tags.latest` and builds a version timeline from `data.time` entries.

#### Applying results

Once a `latestVersion` (and optional `versionTimeline`) is found, all `allResults` entries matching that package are updated:
```js
r.latestVersion = latestVersion;
r.allVersions = versionTimeline;
r.releases = releases;        // GitHub release objects (for changelog)
r.diffType = semverDiffType(r.version, latestVersion);
```

---

## Semver Comparison & Diff Classification

### `parseSemver(v)`
Strips any leading non-digit characters (e.g. `v`, `^`, `~`), then splits on `.` and parses the first three segments as integers. Returns `{ major, minor, patch }`.

### `compareSemver(a, b)`
Returns negative / zero / positive — suitable for use as a sort comparator.

### `semverDiffType(current, latest)`
Compares the two versions and returns one of five string labels:

| Label | Condition |
|---|---|
| `current` | `current` ≥ `latest` |
| `patch` | Same major and minor; latest has higher patch |
| `minor` | Same major; latest has higher minor |
| `major` | Latest has higher major |
| `unknown` | `latestVersion` is `null` / unresolvable |

These labels drive badge colors in the table and segment colors in the freshness chart.

---

## Version Timeline Construction

### From GitHub tags — `buildTagTimeline(componentName, allTags, releases)`

For a given component, iterates all tags and extracts the version number using the same three regex patterns as Phase 2 Step 4. Matches a GitHub release to each tag for date and body text. Returns an array sorted ascending by semver.

### In the detail panel — `toggleDetail(idx, btn)`

Filters the component's `allVersions` to only those between the installed version (exclusive) and the latest (inclusive). Truncates to the most recent 10 for display. For each version:
- Shows version number, release date, and up to 10 lines of release notes (markdown bullet/header lines are lightly converted to HTML `<li>` / `<strong>` elements; all `<` and `>` characters are escaped).
- The current version gets an amber dot; the latest gets a green dot + `LATEST` badge.

---

## AbortController Cancellation

A single `AbortController` is created at the start of `startScan()`. Its `signal` is threaded through every `ghFetch` and `fetchNpmLatest` call made during the scan (including all Phase 0/1/2 calls).

When the user clicks **Cancel**, `scanAbort.abort()` is called. Any pending `fetch` call that holds the signal will reject with a `DOMException` named `AbortError`.

The scan loop catches `AbortError` at two points:
1. Checked explicitly via `signal.aborted` before each batch in Phase 1 and each package in Phase 2.
2. Re-thrown if encountered inside `scanRepo` subdirectory fetches or the version resolution helpers.

On catch, the status message is set to "Scan cancelled." and the loading state is reset. `scanAbort` is set back to `null` in the `finally` block.

---

## Filtering Logic

`filterTable()` runs on every keystroke (text box) or change (dropdowns). It resets `currentPage = 1` then rebuilds `filteredResults`:

```js
filteredResults = allResults.filter(r => {
  if (product && productMap[r.repo] !== product) return false;
  if (repoClass && repoClassMap[r.repo] !== repoClass) return false;
  if (q) {
    const text = (r.repo + ' ' + r.name + ' ' + r.file + ' ' + r.version
                + ' ' + (r.latestVersion || '') + ' ' + r.depType).toLowerCase();
    if (!text.includes(q)) return false;
  }
  return true;
});
```

All three conditions are AND-combined. The text filter matches against repo name, full package name, file path, installed version, latest version, and dependency type.

`renderTable(filteredResults)` is called with the filtered array immediately after.

---

## Table Rendering & Sort Order

`renderTable(results)` sorts its input in place before rendering:

```
primary sort:   a.repo.localeCompare(b.repo)     (alphabetical by repo)
secondary sort: diffOrder[a.diffType]             (major=0, minor=1, patch=2, unknown=3, current=4)
tertiary sort:  a.name.localeCompare(b.name)      (alphabetical by package name)
```

Group header rows are injected whenever the repo name changes while iterating over the page slice. The full `repoCountMap` (counts across _all_ filtered results, not just the current page) is pre-computed so the header shows the true total even when the page only shows a partial group.

---

## Pagination Logic

Page slice calculation:
```js
const startIdx = (currentPage - 1) * pageSize;
const endIdx   = Math.min(startIdx + pageSize, results.length);
const pageResults = results.slice(startIdx, endIdx);
```

The pagination toolbar is only rendered when `results.length > 25`.

Page button construction targets a maximum of **7 visible page buttons**. For large page counts, an ellipsis strategy is applied:
- Always show page 1 and the last page.
- Show a window of 5 pages centred on `currentPage` (`currentPage − 2` to `currentPage + 2`), clamped at the edges.
- Insert `...` (a disabled button) wherever there is a gap between shown pages.

`changePage(page)` updates `currentPage`, re-renders the table, and smooth-scrolls the table card into view.

`changePageSize(size)` resets `currentPage = 1`, updates `pageSize`, and re-renders.
