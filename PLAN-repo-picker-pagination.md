# Plan: Multi-Select Repo Picker + Table Pagination

## Project Context
- **Repo**: `ajay-sajwan-vectorsolutions/local-assets` — hosted on GitHub Pages
- **App**: Single-file HTML dashboard (`index.html`) that audits `@vector-web-components` usage across the **VectorLearning** GitHub org
- **Stack**: Pure HTML/CSS/JS + Chart.js (CDN). No build tools or frameworks.
- **Current state**: Working dashboard with config panel, Chart.js visualizations (doughnut + stacked bar), detailed results table with expandable "View changes" panel, version resolution from GitHub lib repo + npm fallback
- **Hardcoded values**: Org = `VectorLearning`, Component lib repo = `VectorLearning/vector-web-components` (both non-editable labels)
- **Default scan repos**: `ts-react`, `eval-pd-spa`, `lti-react`

## Problem
Currently the repos-to-scan field is a plain text input with comma-separated values. The user wants a proper multi-select dropdown that dynamically fetches all org repos from GitHub, lets the user pick which ones to scan, and includes a "Select All" option. The table also needs pagination to handle large result sets. Additionally, scanning 250+ repos sequentially takes ~5 minutes — needs optimization.

## Approach
Build a **custom multi-select dropdown** in pure HTML/CSS/JS (no extra library — keeps it self-contained alongside Chart.js). Add **table pagination** for the results.

## Changes (all in `index.html`)

### 1. Replace repo text input with a multi-select widget

**HTML** (replace the current `repoList` form-group):
- A styled container with:
  - A **"Load Repos" button** — fetches all repos from the VectorLearning org via GitHub API (requires PAT to be entered first)
  - A trigger bar showing selected count / chip tags
  - A dropdown panel (hidden by default) containing:
    - Search/filter input at the top
    - **"Select All" / "Deselect All"** toggle button
    - Scrollable checkbox list of repo names (max-height with overflow scroll)
  - The 3 default repos (`ts-react`, `eval-pd-spa`, `lti-react`) pre-checked after load

**CSS** additions:
| Class | Purpose |
|-------|---------|
| `.repo-picker` | Outer wrapper (relative positioned) |
| `.repo-picker-trigger` | Clickable bar (looks like a form input, shows chips or count) |
| `.repo-picker-dropdown` | Absolute positioned panel below the trigger |
| `.repo-picker-search` | Filter input inside dropdown |
| `.repo-picker-list` | Scrollable list container (`max-height: 260px; overflow-y: auto`) |
| `.repo-chip` | Small tag for selected repos (shown when <=5 selected; otherwise "N repos selected") |
| `.repo-item` | Individual checkbox row with hover highlight |

**JS** additions:
| Function | Purpose |
|----------|---------|
| `loadOrgRepos()` | Calls GitHub API to list all org repos, populates the dropdown checkboxes |
| `toggleRepoPicker()` | Open/close dropdown |
| `toggleSelectAll()` | Check/uncheck all visible (filtered) repos |
| `filterRepos()` | Filter the checkbox list by search text |
| `getSelectedRepos()` | Returns array of selected repo names (used by `startScan()`) |
| Outside click handler | Closes dropdown when clicking elsewhere |

### 2. Scan performance optimizations (~250 repos → seconds instead of minutes)

**Problem**: Scanning 250 repos sequentially = ~1,000 API calls = ~5 minutes.

**Three optimizations combined**:

| Optimization | How | Impact |
|---|---|---|
| **A. GitHub Code Search pre-filter** | Before deep-scanning, call `GET /search/code?q=vector-web-components+filename:package.json+org:VectorLearning` | Identifies the ~10-20 repos that actually use the library. Eliminates scanning the other 230+ repos entirely. |
| **B. Concurrent scanning** | Scan repos in parallel batches (5-10 at a time) using `Promise.all` with a concurrency limiter | Reduces wall-clock time by 5-10x for the remaining deep-scans. |
| **C. Skip unnecessary monorepo checks** | After reading root `package.json`, only check `packages/apps/libs` dirs if a `workspaces` field exists | Saves ~3 API calls per non-monorepo (most repos). |

**JS changes**:
| Function | Purpose |
|----------|---------|
| `preFilterWithSearch(org, token)` | Uses GitHub Code Search API to find repos containing `@vector-web-components` in `package.json` files. Returns set of repo names. |
| `runConcurrent(tasks, limit)` | Generic concurrency limiter — runs async functions N at a time |
| Updated `scanRepo()` | Checks for `workspaces` field before scanning subdirs |
| Updated `startScan()` | Integrates pre-filter → only deep-scans matching repos, runs concurrently |

**Flow in `startScan()`**:
1. Get selected repos from picker
2. Call Code Search to identify which of those repos contain `@vector-web-components`
3. Deep-scan only the matching repos (concurrent, 5 at a time)
4. Resolve versions from lib repo / npm

**Estimated result for 250 repos**: ~15-30 seconds total.

### 3. Update `startScan()` flow
- Replace `document.getElementById('repoList').value` parsing with `getSelectedRepos()`
- Remove the "fetch all org repos if empty" fallback — repos are now explicitly selected via the picker
- If no repos selected, show an error status message
- Integrate Code Search pre-filter and concurrent scanning (see optimization section above)

### 4. Add table pagination
- Add a pagination toolbar below the table:
  - Row count: "Showing 1–25 of 142 results"
  - Page size selector: dropdown with 25 / 50 / 100 options
  - Prev / page numbers / Next buttons
- `renderTable()` only renders the current page slice of results
- Store pagination state: `currentPage`, `pageSize`
- `changePage(n)` and `changePageSize(n)` functions
- Existing filter input resets to page 1 when typing

## User Flow
```
1. User enters GitHub PAT
2. Clicks "Load Repos" button
3. All VectorLearning org repos appear in multi-select dropdown
4. User picks repos (or clicks "Select All")
5. Clicks "Scan Repos"
6. Results table shows with pagination if many rows
```

## Files Modified
- `index.html` — CSS, HTML, and JS sections (single-file app)

## Verification
1. Open `index.html` in a browser
2. Enter a GitHub PAT
3. Click "Load Repos" — confirm all VectorLearning org repos appear in the dropdown
4. Use search to filter repos, use "Select All", pick individual repos
5. Click "Scan Repos" — confirm:
   - Code Search pre-filter runs first ("Pre-filtering repos..." status)
   - Only repos with actual `@vector-web-components` usage are deep-scanned
   - Scanning completes in seconds, not minutes
6. Verify the results table shows pagination controls when results exceed page size
7. Test page navigation, page size change, and filter + pagination interaction
