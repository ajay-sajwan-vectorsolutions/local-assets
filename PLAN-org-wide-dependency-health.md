# Plan: Org-Wide Dependency Health Platform

## Status: READY TO IMPLEMENT

## Overview

Transform the single-purpose VWC dashboard into a full org-wide dependency health tool — supporting npm, Maven, NuGet, Python, and Ruby ecosystems with CVE scanning and a management-facing Health Summary view.

---

## Files to Modify

| File | Changes |
|---|---|
| `index.html` | All feature implementation — CSS, HTML, JS |
| `CLAUDE.md` | Update "What This Is" description |

---

## Phase 1 Features (implement together)

1. Configurable Package Scope
2. Multi-Ecosystem Manifest Scanner
3. Security Advisory / CVE Scanning
4. Repo Health Summary View

---

## Traceability — Implementation Progress

> Update status column as work progresses: `pending` → `in progress` → `done`
> Re-count the summary row after each session.

### Summary

| Category | Total Items | Done | In Progress | Pending |
|---|---|---|---|---|
| CSS | 5 | 0 | 0 | 5 |
| HTML — Config Card | 3 | 0 | 0 | 3 |
| HTML — Results Area | 5 | 0 | 0 | 5 |
| JS — Helpers & Parsers | 9 | 0 | 0 | 9 |
| JS — Scan Logic (startScan) | 5 | 0 | 0 | 5 |
| JS — Rendering | 8 | 0 | 0 | 8 |
| JS — Filtering & Export | 3 | 0 | 0 | 3 |
| Other | 2 | 0 | 0 | 2 |
| **TOTAL** | **40** | **0** | **0** | **40** |

**Overall completion: 0 / 40 (0%)**

---

### CSS (5 items)

| # | Item | Location in file | Status |
|---|---|---|---|
| C1 | Add `.badge-dark-red` (VULNERABLE badge) | Before `</style>` | pending |
| C2 | Add `.ecosystem-badge` (Type column pill) | Before `</style>` | pending |
| C3 | Add `.freshness-bar` / `.freshness-fill` (Health Summary) | Before `</style>` | pending |
| C4 | Add `.ecosystems-group` / `.ecosystem-check-label` (config checkboxes) | Before `</style>` | pending |
| C5 | Add `.cve-list` / `.cve-item` / `.cve-severity-*` (CVE detail panel) | Before `</style>` | pending |

**CSS sub-total: 0 / 5**

---

### HTML — Config Card (3 items)

| # | Item | Location in file | Status |
|---|---|---|---|
| H1 | Add "Packages to Audit" text input (`#packageScopeInput`) | After Component Library Repo field | pending |
| H2 | Add Ecosystems checkboxes (`#ecoNpm`, `#ecoMaven`, `#ecoNuget`, `#ecoPython`, `#ecoRuby`) | After Packages to Audit field | pending |
| H3 | Add "Check for CVEs" checkbox (`#enableCVECheck`) | Between `.form-grid` close and `.actions` | pending |

**HTML Config sub-total: 0 / 3**

---

### HTML — Results Area (5 items)

| # | Item | Location in file | Status |
|---|---|---|---|
| H4 | Add 5th metric card `#metricCVEs` (Security Advisories) | After Upgrades Available card | pending |
| H5 | Add `#effortCVEs` span to effort banner | Before `#effortRepos` in effort banner | pending |
| H6 | Add "Health Summary" view toggle button (`#viewHealth`) | In `.view-toggle` div | pending |
| H7 | Add ecosystem filter select (`#filterEcosystem`) to toolbar | Before `#filterStatus` in toolbar | pending |
| H8 | Add `Vulnerable (CVE)` option to `#filterStatus`; add IDs to `<th>` elements; update no-results text | Table toolbar + table headers | pending |

**HTML Results sub-total: 0 / 5**

---

### JS — Helpers & Parsers (9 items)

| # | Item | Replaces / Adds | Status |
|---|---|---|---|
| J1 | Update page `<title>` and `<h1>` / `<p>` header text | n/a — HTML text change | pending |
| J2 | `matchesPatterns()` + `extractMatchingDependencies()` | Replaces `extractVectorComponents()` | pending |
| J3 | `fetchRawFileContent()` — returns raw text for XML/text files | New function after `fetchFileContent` | pending |
| J4 | `extractMavenDeps(xmlText)` | New function | pending |
| J5 | `extractNugetDeps(xmlText)` | New function | pending |
| J6 | `extractPythonDeps(reqText)` | New function | pending |
| J7 | `extractRubyDeps(lockText)` | New function | pending |
| J8 | `resolveLatestMaven()` / `resolveLatestNuget()` / `resolveLatestPypi()` / `resolveLatestRubyGems()` | 4 new version resolver functions | pending |
| J9 | `isVersionVulnerable(installedVersion, rangeStr)` | New CVE range checker | pending |

**JS Helpers sub-total: 0 / 9**

---

### JS — Scan Logic (5 items)

| # | Item | Replaces / Adds | Status |
|---|---|---|---|
| S1 | Update `scanRepo()` — new signature + per-ecosystem manifest scanning | Replaces current `scanRepo` | pending |
| S2 | Update `startScan()` Phase 0 — read DOM config; parallel ecosystem code search queries | Replaces Phase 0 block in `startScan` | pending |
| S3 | Update `startScan()` Phase 1 — pass `ecosystems` + `patterns` to `scanRepo()` | Modify batch loop in `startScan` | pending |
| S4 | Update `startScan()` Phase 2 — per-ecosystem version resolution branch | Replaces Phase 2 block in `startScan` | pending |
| S5 | Add `startScan()` Phase 3 — CVE scan via `checkSecurityAdvisories()` | New phase added after Phase 2 | pending |

**JS Scan sub-total: 0 / 5**

---

### JS — Rendering (8 items)

| # | Item | Replaces / Adds | Status |
|---|---|---|---|
| R1 | `checkSecurityAdvisories(results, token, signal)` | New function | pending |
| R2 | Update `statusBadge(r)` — add VULNERABLE case at top | Modify existing function | pending |
| R3 | Update `renderDashboard()` — populate `#metricCVEs` | Modify existing function | pending |
| R4 | Update `renderEffortBanner()` — add CVE count to banner | Modify existing function | pending |
| R5 | Update `renderTable()` — sort order, package display name, ecosystem badge in Type cell | Modify existing function | pending |
| R6 | Update `toggleDetail()` — prepend CVE advisory panel when `r.advisories` present | Modify existing function | pending |
| R7 | New `renderHealthTable()` — one row per repo with freshness bar, CVE count, last commit | New async function | pending |
| R8 | Update `setPivotView()` — add 'health' mode, swap table headers, hide/show toolbar | Modify existing function | pending |

**JS Rendering sub-total: 0 / 8**

---

### JS — Filtering & Export (3 items)

| # | Item | Replaces / Adds | Status |
|---|---|---|---|
| F1 | Update `filterTable()` — add ecosystem filter + vulnerable status handling | Modify existing function | pending |
| F2 | Update `exportCSV()` — add Ecosystem + CVEs columns | Modify existing function | pending |
| F3 | Update `exportJSON()` — add ecosystem, cveCount, advisories fields | Modify existing function | pending |

**JS Filtering sub-total: 0 / 3**

---

### Other (2 items)

| # | Item | Status |
|---|---|---|
| O1 | Update `CLAUDE.md` — revise "What This Is" description to reflect expanded scope | pending |
| O2 | Update `.metrics` CSS grid: `repeat(4, 1fr)` → `repeat(auto-fit, minmax(180px, 1fr))` | pending |

**Other sub-total: 0 / 2**

---

## Section-by-Section Changes

### 1. `<head>` / Header

| Location | Old | New |
|---|---|---|
| `<title>` | `Vector Web Components Dashboard` | `VectorLearning Dependency Health` |
| `<h1>` | `Vector Web Components Dashboard` | `VectorLearning Dependency Health` |
| `<p>` | `Scan GitHub repos to audit @vector-web-components usage...` | `Audit dependency health across all org repos — versions, CVEs, and upgrade effort` |

---

### 2. New CSS (add before `</style>`)

```css
/* VULNERABLE badge */
.badge-dark-red {
  background: #450a0a;
  color: #fca5a5;
  border: 1px solid #7f1d1d;
}

/* Ecosystem badge (shown in Type column) */
.ecosystem-badge {
  display: inline-flex;
  align-items: center;
  padding: 2px 7px;
  border-radius: 4px;
  font-size: 0.72rem;
  font-weight: 600;
  background: #f1f5f9;
  color: #475569;
  border: 1px solid #e2e8f0;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

/* Health summary freshness bar */
.freshness-bar {
  height: 8px; border-radius: 4px; background: var(--border);
  overflow: hidden; width: 80px; display: inline-block;
  vertical-align: middle; margin-right: 6px;
}
.freshness-fill { height: 100%; border-radius: 4px; }

/* Ecosystems checkboxes in config */
.ecosystems-group { display: flex; flex-wrap: wrap; gap: 10px; align-items: center; margin-top: 4px; }
.ecosystem-check-label { display: inline-flex; align-items: center; gap: 5px; font-size: 0.85rem; color: var(--text); cursor: pointer; }
.ecosystem-check-label input[type="checkbox"] { accent-color: var(--primary); width: 14px; height: 14px; cursor: pointer; }

/* CVE advisory detail panel */
.cve-list { display: flex; flex-direction: column; gap: 8px; margin-top: 8px; }
.cve-item { background: #fff5f5; border: 1px solid #fecaca; border-radius: 6px; padding: 10px 14px; font-size: 0.82rem; }
.cve-item .cve-id { font-weight: 700; color: #991b1b; }
.cve-severity { font-size: 0.72rem; font-weight: 600; padding: 1px 6px; border-radius: 10px; margin-left: 6px; }
.cve-severity-critical { background: #450a0a; color: #fca5a5; }
.cve-severity-high { background: #fef2f2; color: #991b1b; border: 1px solid #fecaca; }
.cve-severity-medium { background: #fffbeb; color: #92400e; border: 1px solid #fde68a; }
.cve-severity-low { background: #f0fdf4; color: #15803d; border: 1px solid #bbf7d0; }
```

Also update `.metrics` grid: `repeat(4, 1fr)` → `repeat(auto-fit, minmax(180px, 1fr))`

---

### 3. Config Card HTML Additions

After the **Component Library Repo** field, inside `.form-grid`, add two new full-width rows:

**Row 1 — Packages to Audit:**
```html
<div class="form-group form-full">
  <label>Packages to Audit
    <span style="font-weight:400;text-transform:none;font-size:0.72rem;color:var(--text-muted);">
      (comma-separated; use * for all packages)
    </span>
  </label>
  <input type="text" id="packageScopeInput" value="@vector-web-components/*"
    placeholder="@vector-web-components/*, react, axios" />
  <span class="help-text">
    Default: <code>@vector-web-components/*</code> — audits VWC only.
    Set <code>*</code> to scan all packages (skips Phase 0 pre-filter).
  </span>
</div>
```

**Row 2 — Ecosystems:**
```html
<div class="form-group form-full">
  <label>Ecosystems</label>
  <div class="ecosystems-group">
    <label class="ecosystem-check-label">
      <input type="checkbox" id="ecoNpm" checked onchange="updateEcosystems()"> npm
    </label>
    <label class="ecosystem-check-label">
      <input type="checkbox" id="ecoMaven" onchange="updateEcosystems()"> Maven (pom.xml)
    </label>
    <label class="ecosystem-check-label">
      <input type="checkbox" id="ecoNuget" onchange="updateEcosystems()"> NuGet (.csproj)
    </label>
    <label class="ecosystem-check-label">
      <input type="checkbox" id="ecoPython" onchange="updateEcosystems()"> Python (requirements.txt)
    </label>
    <label class="ecosystem-check-label">
      <input type="checkbox" id="ecoRuby" onchange="updateEcosystems()"> Ruby (Gemfile.lock)
    </label>
  </div>
</div>
```

**CVE checkbox** — add between `.form-grid` close and `.actions`:
```html
<div style="margin-bottom:12px;display:flex;align-items:center;gap:12px;">
  <label class="ecosystem-check-label">
    <input type="checkbox" id="enableCVECheck">
    <span>Check for CVEs
      <span style="font-weight:400;font-size:0.78rem;color:var(--text-muted);">
        (slower — queries GitHub Security Advisory DB)
      </span>
    </span>
  </label>
</div>
```

---

### 4. Metrics Row HTML

Add 5th metric card (after the Upgrades Available card):
```html
<div class="metric-card red hidden" id="metricCVEs">
  <div class="label">Security Advisories</div>
  <div class="value">0</div>
  <div class="detail">vulnerable packages</div>
</div>
```

---

### 5. Effort Banner HTML

Add CVE effort item before `effortRepos` span:
```html
<span class="effort-item hidden" id="effortCVEs"></span>
```

---

### 6. Table Card Header

**View toggle** — add Health Summary button:
```html
<button class="view-toggle-btn" id="viewHealth" onclick="setPivotView('health')">Health Summary</button>
```

**Toolbar** — add ecosystem filter select (before filterStatus):
```html
<select class="toolbar-select" id="filterEcosystem" onchange="filterTable()">
  <option value="">All Ecosystems</option>
  <option value="npm">npm</option>
  <option value="maven">Maven</option>
  <option value="nuget">NuGet</option>
  <option value="python">Python</option>
  <option value="ruby">Ruby</option>
</select>
```

**Status filter** — add Vulnerable option at top:
```html
<option value="vulnerable">Vulnerable (CVE)</option>
```

**Table `<th>` elements** — add IDs to all headers (for health summary header swapping):
- `<th id="thInstalled">Installed</th>`
- `<th id="thLatest">Latest</th>`
- `<th id="thStatus">Status</th>`
- `<th id="thType">Type</th>`
- `<th id="thDetails">Details</th>`

**No-results text**: Change `No @vector-web-components found in the scanned repos.` → `No dependencies found in the scanned repos.`

---

## JavaScript Changes

### New State Variables (after existing state block)

```js
// Scan config state (read fresh from DOM in startScan)
// No persistent JS state needed — values read from inputs on each scan
```

---

### New Helper Functions

#### `matchesPatterns(name, patterns)` + `extractMatchingDependencies()`
Replaces `extractVectorComponents()`:

```js
function matchesPatterns(name, patterns) {
  if (!patterns || patterns.length === 0 || patterns.includes('*')) return true;
  return patterns.some(p => {
    const pat = p.trim();
    if (!pat || pat === '*') return true;
    if (pat.endsWith('/*')) return name.startsWith(pat.slice(0, -1));
    return name === pat;
  });
}

function extractMatchingDependencies(pkgJson, patterns, depType) {
  const deps = pkgJson[depType];
  if (!deps) return [];
  return Object.entries(deps)
    .filter(([name]) => matchesPatterns(name, patterns))
    .map(([name, version]) => ({ name, version, depType, ecosystem: 'npm' }));
}
```

#### `fetchRawFileContent()`
Returns raw text (for XML/text manifests):

```js
async function fetchRawFileContent(fullRepoPath, filePath, token, signal) {
  const url = `https://api.github.com/repos/${fullRepoPath}/contents/${filePath}`;
  const data = await ghFetch(url, token, signal);
  if (!data || !data.content) return null;
  try { return atob(data.content.replace(/\n/g, '')); }
  catch { return null; }
}
```

#### Ecosystem Parsers

```js
function extractMavenDeps(xmlText) {
  const deps = [];
  const depBlockRegex = /<dependency>([\s\S]*?)<\/dependency>/g;
  let match;
  while ((match = depBlockRegex.exec(xmlText)) !== null) {
    const block = match[1];
    const groupId = (block.match(/<groupId>([^<]+)<\/groupId>/) || [])[1];
    const artifactId = (block.match(/<artifactId>([^<]+)<\/artifactId>/) || [])[1];
    const version = (block.match(/<version>([^<]+)<\/version>/) || [])[1];
    const scope = (block.match(/<scope>([^<]+)<\/scope>/) || [])[1] || 'compile';
    if (groupId && artifactId)
      deps.push({ name: `${groupId}:${artifactId}`, version: version || 'unknown', depType: scope, ecosystem: 'maven' });
  }
  return deps;
}

function extractNugetDeps(xmlText) {
  const deps = [];
  const pkgRefRegex = /<PackageReference\s+Include="([^"]+)"(?:[^>]*Version="([^"]*)")?[^>]*(?:\/>|>[\s\S]*?<\/PackageReference>)/gi;
  let match;
  while ((match = pkgRefRegex.exec(xmlText)) !== null) {
    let version = match[2];
    if (!version) {
      const after = xmlText.slice(match.index, match.index + 200);
      const vc = after.match(/<Version>([^<]+)<\/Version>/i);
      if (vc) version = vc[1];
    }
    if (match[1]) deps.push({ name: match[1], version: version || 'unknown', depType: 'dependency', ecosystem: 'nuget' });
  }
  const pkgConfigRegex = /<package\s+id="([^"]+)"\s+version="([^"]+)"/gi;
  while ((match = pkgConfigRegex.exec(xmlText)) !== null)
    deps.push({ name: match[1], version: match[2], depType: 'dependency', ecosystem: 'nuget' });
  return deps;
}

function extractPythonDeps(reqText) {
  const deps = [];
  for (const line of reqText.split('\n')) {
    const t = line.trim();
    if (!t || t.startsWith('#') || t.startsWith('-')) continue;
    const m = t.match(/^([A-Za-z0-9_.\-]+)\s*(?:==|>=|~=|<=|!=|>|<)\s*([^\s,;]+)/);
    if (m) deps.push({ name: m[1], version: m[2], depType: 'dependency', ecosystem: 'python' });
    else if (/^[A-Za-z0-9_.\-]+$/.test(t))
      deps.push({ name: t, version: 'unknown', depType: 'dependency', ecosystem: 'python' });
  }
  return deps;
}

function extractRubyDeps(lockText) {
  const deps = [];
  const gemRegex = /^    ([a-zA-Z0-9_\-\.]+) \((\d[^)]*)\)$/gm;
  let match;
  while ((match = gemRegex.exec(lockText)) !== null)
    deps.push({ name: match[1], version: match[2], depType: 'dependency', ecosystem: 'ruby' });
  return deps;
}
```

#### Version Resolvers (per ecosystem)

```js
async function resolveLatestMaven(name, signal) {
  const [groupId, artifactId] = name.split(':');
  if (!groupId || !artifactId) return null;
  try {
    const url = `https://search.maven.org/solrsearch/select?q=g:${encodeURIComponent(groupId)}+AND+a:${encodeURIComponent(artifactId)}&wt=json&rows=1`;
    const resp = await fetch(url, signal ? { signal } : {});
    if (!resp.ok) return null;
    const data = await resp.json();
    return data?.response?.docs?.[0]?.latestVersion || data?.response?.docs?.[0]?.v || null;
  } catch { return null; }
}

async function resolveLatestNuget(name, signal) {
  try {
    const url = `https://api.nuget.org/v3-flatcontainer/${name.toLowerCase()}/index.json`;
    const resp = await fetch(url, signal ? { signal } : {});
    if (!resp.ok) return null;
    const data = await resp.json();
    const versions = data?.versions || [];
    const stable = versions.filter(v => !v.includes('-'));
    return stable.length ? stable[stable.length - 1] : (versions.length ? versions[versions.length - 1] : null);
  } catch { return null; }
}

async function resolveLatestPypi(name, signal) {
  try {
    const url = `https://pypi.org/pypi/${encodeURIComponent(name)}/json`;
    const resp = await fetch(url, signal ? { signal } : {});
    if (!resp.ok) return null;
    const data = await resp.json();
    return data?.info?.version || null;
  } catch { return null; }
}

async function resolveLatestRubyGems(name, signal) {
  try {
    const url = `https://rubygems.org/api/v1/gems/${encodeURIComponent(name)}.json`;
    const resp = await fetch(url, signal ? { signal } : {});
    if (!resp.ok) return null;
    const data = await resp.json();
    return data?.version || null;
  } catch { return null; }
}
```

---

### Updated `scanRepo(org, repo, token, signal, ecosystems, patterns)`

Replaces current `scanRepo`. New signature adds `ecosystems` (Set) and `patterns` (Array) params.

```
- npm:    existing logic using extractMatchingDependencies() instead of extractVectorComponents()
- maven:  fetchRawFileContent pom.xml → extractMavenDeps()
- nuget:  list root dir → find *.csproj files + packages.config → extractNugetDeps()
- python: fetchRawFileContent requirements.txt → extractPythonDeps()
- ruby:   fetchRawFileContent Gemfile.lock → extractRubyDeps()
```

All results gain `ecosystem` field: `'npm' | 'maven' | 'nuget' | 'python' | 'ruby'`

---

### Updated `startScan()` — 4 Phases

#### Reads from DOM at start:
```js
const rawPatterns = (document.getElementById('packageScopeInput').value || '@vector-web-components/*')
  .split(',').map(s => s.trim()).filter(Boolean);
const isWildcard = rawPatterns.length === 0 || rawPatterns.includes('*');
const ecosystems = new Set([
  ...document.getElementById('ecoNpm').checked ? ['npm'] : [],
  ...document.getElementById('ecoMaven').checked ? ['maven'] : [],
  ...document.getElementById('ecoNuget').checked ? ['nuget'] : [],
  ...document.getElementById('ecoPython').checked ? ['python'] : [],
  ...document.getElementById('ecoRuby').checked ? ['ruby'] : [],
]);
const runCVE = document.getElementById('enableCVECheck').checked;
```

#### Phase 0 — Parameterised Code Search (parallel)
- Build one query per ecosystem + per pattern (for npm)
- Run all queries via `Promise.allSettled` (parallel)
- Queries:
  - npm + specific pattern: `"${term}" filename:package.json org:${org}`
  - npm + wildcard: `filename:package.json org:${org}`
  - maven: `filename:pom.xml org:${org}`
  - nuget: `filename:.csproj org:${org}`
  - python: `filename:requirements.txt org:${org}`
  - ruby: `filename:Gemfile.lock org:${org}`
- If all queries return 0 hits, fall back to scanning all selected repos

#### Phase 1 — Concurrent Scan (batches of 5)
- Pass `ecosystems` and `rawPatterns` to `scanRepo()`
- No other changes from current logic

#### Phase 2 — Version Resolution (per ecosystem)
- Build unique `(ecosystem, name)` pairs from `allResults`
- Branch by ecosystem:
  - `npm` → existing lib repo + npm registry logic
  - `maven` → `resolveLatestMaven(name, signal)`
  - `nuget` → `resolveLatestNuget(name, signal)`
  - `python` → `resolveLatestPypi(name, signal)`
  - `ruby` → `resolveLatestRubyGems(name, signal)`

#### Phase 3 — CVE Scan (opt-in)
```
if (runCVE && allResults.length > 0):
  call checkSecurityAdvisories(allResults, token, signal)
  sets r.advisories = [...] and r.diffType = 'vulnerable' on matches
```

---

### New `checkSecurityAdvisories(results, token, signal)`

```
API: GET https://api.github.com/advisories?type=reviewed&ecosystem=<ghEco>&package=<name>&per_page=100
Ecosystem map: { npm→npm, maven→maven, nuget→nuget, python→pip, ruby→rubygems }

For each unique (ecosystem, name):
  fetch advisories
  for each result matching that (ecosystem, name):
    check isVersionVulnerable(r.version, advisory.vulnerable_version_range)
    if vulnerable: push to r.advisories[], set r.diffType = 'vulnerable'
```

### New `isVersionVulnerable(installedVersion, rangeStr)`
Parses range strings like `>= 1.0.0, < 2.0.0` using existing `compareSemver()`.

---

### Updated Rendering Functions

#### `statusBadge(r)` — add VULNERABLE at top:
```js
if (r.advisories?.length > 0 || r.diffType === 'vulnerable')
  return '<span class="badge badge-dark-red">⚠ VULNERABLE</span>';
```

#### `renderTable(results)` — changes:
- Sort order: add `vulnerable: -1` to diffOrder (sorts above major)
- Package display: `r.ecosystem === 'npm' ? r.name.replace('@vector-web-components/', '') : r.name`
- Type cell: `<span class="ecosystem-badge">${r.ecosystem||'npm'}</span> <span class="dep-type">${r.depType}</span>`
- File link: uses `r.ecosystem !== 'npm'` to skip VWC prefix in display

#### `toggleDetail(idx, btn)` — add CVE panel before version timeline:
```
if (r.advisories?.length > 0):
  render .cve-list with .cve-item per advisory
  show GHSA ID, CVE ID, severity badge, summary, patched version
```

#### `renderDashboard(results, repoCount, errors)` — changes:
```js
const vulnCount = results.filter(r => r.advisories?.length > 0).length;
document.querySelector('#metricCVEs .value').textContent = vulnCount;
document.getElementById('metricCVEs').classList.toggle('hidden', vulnCount === 0);
```

#### `renderEffortBanner(results)` — add CVE line:
```js
const vuln = results.filter(r => r.advisories?.length > 0).length;
document.getElementById('effortCVEs').innerHTML = vuln > 0
  ? `<span style="color:#7f1d1d">▼</span> ${vuln} vulnerable` : '';
document.getElementById('effortCVEs').classList.toggle('hidden', vuln === 0);
```

---

### New `renderHealthTable()` (async)

Called when `pivotView === 'health'`. Steps:
1. Compute per-repo stats from `allResults`:
   - `total`, `upToDate`, `outdated`, `vulnerable`, `ecosystems` (Set), `cveCount`
2. Fetch last commit per repo in parallel (one API call each):
   `GET /repos/${org}/${repo}/commits?per_page=1`
3. Render one `<tr>` per repo, sorted by freshness ascending (least fresh first)

Columns:
| Column | Source |
|---|---|
| Repo | Link to GitHub |
| Product | `productMap[repo]` |
| Freshness | bar + % `(upToDate/total*100)` — green ≥80%, amber 50–79%, red <50% |
| CVEs | badge count or `—` |
| Ecosystems | Set of ecosystems found |
| Last Commit | date from API |
| (empty) | — |

Also swaps table header text:
```js
document.getElementById('thFile').textContent = 'Repo';
document.getElementById('thPackage').textContent = 'Product';
document.getElementById('thInstalled').textContent = 'Freshness';
document.getElementById('thLatest').textContent = 'CVEs';
document.getElementById('thStatus').textContent = 'Ecosystems';
document.getElementById('thType').textContent = 'Last Commit';
document.getElementById('thDetails').textContent = '';
```

---

### Updated `setPivotView(view)` — add 'health' mode:
```js
document.getElementById('viewHealth').classList.toggle('active', view === 'health');
if (view === 'health') {
  document.getElementById('tableFilter').closest('.table-toolbar').classList.add('hidden');
  document.getElementById('paginationToolbar').classList.add('hidden');
  renderHealthTable(); // async, fires and forgets
} else {
  document.getElementById('tableFilter').closest('.table-toolbar').classList.remove('hidden');
  // restore headers, re-render table
}
```

---

### Updated `filterTable()` — add ecosystem filter:
```js
const ecosystemFilter = document.getElementById('filterEcosystem').value;
// In the filter function:
if (ecosystemFilter && r.ecosystem !== ecosystemFilter) return false;
// Update status filter to handle 'vulnerable' separately:
if (status === 'vulnerable' && !r.advisories?.length) return false;
if (status && status !== 'vulnerable' && r.diffType !== status) return false;
```

---

### Updated `exportCSV()` / `exportJSON()`

CSV headers add: `Ecosystem`, `CVEs`
JSON fields add: `ecosystem`, `cveCount`, `advisories`

---

### New `updateEcosystems()` handler (no-op, config read at scan time)
Just validates that at least one ecosystem checkbox is checked.

---

## Implementation Notes

1. **`fetchFileContent`** (existing) parses JSON — keep as-is for npm. New `fetchRawFileContent` returns raw string for XML/text formats.

2. **Phase 0 rate limits**: GitHub Code Search = 30 req/min authenticated. 5 parallel ecosystem queries well within limits.

3. **Non-npm package display names**: Don't strip `@vector-web-components/` prefix for non-npm packages. Use full `r.name`.

4. **Chart compatibility**: Charts remain VWC-focused (no changes needed). They display data correctly for any package names. The click-to-filter behavior uses `r.name` directly for non-npm packages.

5. **Health Summary pagination**: Hide pagination toolbar in health view. Show all repos in one table.

6. **Scan history (`saveScanHistory`)**: No changes needed — still saves diffType per result. CVE status (`vulnerable`) is a valid diffType and will be stored.

7. **`colspan="7"`** appears in group headers and detail rows — no column count changes (ecosystem shown in existing Type column), so no colspan changes needed.

8. **`resolveFromGitHubRepo`** — no changes needed. It still works for npm packages from the component lib repo. The `patterns` filtering in `extractMatchingDependencies` handles which npm packages to include.

---

## Verification Checklist

- [ ] Enter PAT → Load Repos → repos appear
- [ ] Default patterns `@vector-web-components/*` → scan `ts-react` → VWC rows appear (backward compat)
- [ ] Change patterns to `react` → scan `ts-react` → react version rows appear
- [ ] Enable Maven → select a Java repo → pom.xml dependencies appear with `maven` ecosystem badge
- [ ] Enable NuGet → select `RV-VectorLearning` → .csproj deps appear
- [ ] Enable CVE → known-vulnerable package shows red VULNERABLE badge
- [ ] CVE expand panel → shows GHSA ID, severity, description, patched version
- [ ] Switch to Health Summary → one row per repo, freshness bar, last commit date
- [ ] Export CSV → ecosystem + CVE columns present
- [ ] Wildcard `*` mode → all packages in selected repos appear, Phase 0 skipped for npm
- [ ] Cancel scan → works correctly across all phases
- [ ] Auto-refresh → still works after multi-ecosystem scan

---

## Estimated Line Count

| Section | Current | After |
|---|---|---|
| CSS | ~400 | ~510 |
| HTML | ~145 | ~190 |
| JS | ~1500 | ~2000 |
| **Total** | **~3065** | **~3700** |
