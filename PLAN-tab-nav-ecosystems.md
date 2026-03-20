# Plan: UI Navigation Reorganization + Missing Ecosystems

## Context

The dashboard has grown from a simple VWC audit tool into a multi-ecosystem dependency health platform. With 9 planned ecosystems, CVE scanning, Health Summary, and more features incoming (License compliance, CI health, Dependabot PRs), the current single-scroll layout is becoming unwieldy:

- Config card grows taller with each ecosystem added (now 5 checkboxes, soon 9)
- "Health Summary" — the management view — is buried 3 clicks deep (scroll → table card → toggle button)
- No logical separation between "setup", "overview", and "detailed results"
- Future features (License, CI health) have no clean home in the current layout

The user asked: *"Since the system is going to grow with different types of project and tech stack nature. Do you think we need to reorganize the page and relook at the layout. shall we introduce menu/navigation or tabs something like this."*

**Answer: Yes — a 4-tab top navigation is the right move.**

---

## Recommended Approach: 4-Tab Top Navigation

```
[⚙ Scan]  [📊 Overview]  [🔍 Details]  [❤ Health]
```

| Tab | Content | Primary User |
|---|---|---|
| **Scan** | Config card (PAT, repos, ecosystems, packages, CVE toggle, scan button, progress) | Everyone |
| **Overview** | 5 metric cards + effort banner + 2 Chart.js charts | Engineers, managers (quick check) |
| **Details** | Full dependency table with By Repo / By Component sub-views + all filters + pagination | Frontend/backend engineers |
| **Health** | Health Summary table — one row per repo (freshness, CVEs, last commit, ecosystems) | Tech leads, managers |

**Why tabs, not sidebar or scroll-sections:**
- Sidebar is overkill for a focused single-file tool — wastes horizontal space
- Scroll-sections don't solve Health Summary discoverability (still have to scroll past config + charts)
- Tabs are lightweight (pure CSS + ~20 lines JS), keep the single-file constraint, and give Health Summary first-class status

**Auto-navigation:** After scan completes, auto-switch to Overview tab. If the user clicks Health mid-scan, allow it (Health renders live as repos complete).

---

## Part A: Tab Navigation Implementation

### HTML Changes

Wrap existing sections into tab panels. The overall structure becomes:

```html
<div class="header">...</div>

<nav class="tab-nav">
  <button class="tab-btn active" data-tab="scan">⚙ Scan</button>
  <button class="tab-btn" data-tab="overview">📊 Overview</button>
  <button class="tab-btn" data-tab="details">🔍 Details</button>
  <button class="tab-btn" data-tab="health">❤ Health</button>
  <!-- badge span for CVE count on Health tab -->
</nav>

<div class="container">
  <!-- TAB: Scan -->
  <div class="tab-panel active" id="tab-scan">
    <!-- existing .card.config-card unchanged -->
  </div>

  <!-- TAB: Overview -->
  <div class="tab-panel" id="tab-overview">
    <!-- #metricsRow (5 cards) -->
    <!-- #effortBanner -->
    <!-- .charts-row (2 Chart.js cards) -->
  </div>

  <!-- TAB: Details -->
  <div class="tab-panel" id="tab-details">
    <!-- .card.table-card with .view-toggle (By Repo | By Component only) -->
    <!-- .table-toolbar (filters) -->
    <!-- <table> -->
    <!-- #paginationToolbar -->
  </div>

  <!-- TAB: Health -->
  <div class="tab-panel" id="tab-health">
    <!-- Health summary table — dedicated panel with #healthBody tbody -->
  </div>
</div>
```

**Key detail:** The `.view-toggle` buttons inside the table card become **2 buttons only** (`By Repo` | `By Component`). Health Summary is removed from that toggle — it now lives in its own tab.

### CSS Changes (~60 new lines)

```css
.tab-nav {
  display: flex;
  border-bottom: 2px solid var(--border);
  background: white;
  position: sticky;
  top: 0;
  z-index: 100;
  padding: 0 40px;
}
.tab-btn {
  padding: 12px 20px;
  border: none;
  background: none;
  font-weight: 500;
  cursor: pointer;
  border-bottom: 3px solid transparent;
  margin-bottom: -2px;
  color: var(--text-muted);
  transition: all 0.15s;
}
.tab-btn.active {
  color: var(--primary);
  border-bottom-color: var(--primary);
}
.tab-badge {
  /* red bubble for CVE count on Health tab */
  background: var(--red);
  color: white;
  border-radius: 10px;
  font-size: 0.7rem;
  padding: 1px 6px;
}
.tab-panel { display: none; }
.tab-panel.active { display: block; }
```

### JS Changes (~30 lines + small edits)

**`switchTab(tabId)` function:**
```js
function switchTab(tabId) {
  document.querySelectorAll('.tab-btn').forEach(b => b.classList.toggle('active', b.dataset.tab === tabId));
  document.querySelectorAll('.tab-panel').forEach(p => p.classList.toggle('active', p.id === `tab-${tabId}`));
  currentTab = tabId;
  if (tabId === 'health' && allResults.length > 0) renderHealthTable();
  localStorage.setItem('vwc-active-tab', tabId);
}
```

**After scan completes** (in `renderDashboard()`): call `switchTab('overview')`.

**CVE badge update**: In `renderDashboard()`, if CVE count > 0, update Health tab badge.

**`setPivotView()` simplification**: Remove the `'health'` case. Only handles `'repo'` and `'component'` modes.

**`renderHealthTable()` move**: Renders into `#healthBody` (dedicated tbody in `#tab-health`). Called when Health tab is activated instead of via `setPivotView('health')`.

**Chart onClick**: Switch to Details tab instead of scrolling to `.table-card`.

---

## Part B: 4 Missing Ecosystems

Four ecosystems identified from the full 270-repo audit not yet implemented:

| Ecosystem | Manifest | Parser | Version API |
|---|---|---|---|
| **ColdFusion/CommandBox** | `box.json` | JSON (`dependencies` object) | `https://www.forgebox.io/api/v1/entry/:slug` |
| **PHP/Composer** | `composer.json` | JSON (`require` + `require-dev` objects) | `https://packagist.org/packages/:vendor/:package.json` |
| **Dart/Flutter** | `pubspec.yaml` | YAML line-by-line regex under `dependencies:` | `https://pub.dev/api/packages/:package` |
| **Gradle** | `build.gradle`, `build.gradle.kts` | Regex — `implementation '...'` / `implementation("...")` | Maven Central (reuses `resolveLatestMaven`) |

**Implementation pattern** (same as existing npm/Maven/NuGet/Python/Ruby):
1. Add checkbox in config card: `id="ecoColdfusion"`, `id="ecoComposer"`, `id="ecoDart"`, `id="ecoGradle"`
2. Add extract functions: `extractColdFusionDeps()`, `extractComposerDeps()`, `extractDartDeps()`, `extractGradleDeps()`
3. Add resolve functions: `resolveLatestForgeBox()`, `resolveLatestPackagist()`, `resolveLatestPubDev()` (Gradle reuses `resolveLatestMaven()`)
4. Wire into `scanRepo()` as additional if-blocks
5. Register in `ECOSYSTEMS` registry for Phase 0 and Phase 2

---

## Part C: Ecosystem Registry Refactor

**Problem**: Every new ecosystem requires changes in 4+ places: `scanRepo()`, Phase 2 version resolution, Phase 0 code search, config card HTML checkbox, and `filterEcosystem` dropdown option. Easy to miss one.

**Solution**: A single `ECOSYSTEMS` registry object:

```js
const ECOSYSTEMS = {
  npm:        { label: 'npm',        phase0: 'filename:package.json' },
  maven:      { label: 'Maven',      phase0: 'filename:pom.xml',           resolve: resolveLatestMaven },
  nuget:      { label: 'NuGet',      phase0: 'filename:.csproj',           resolve: resolveLatestNuget },
  python:     { label: 'Python',     phase0: 'filename:requirements.txt',  resolve: resolveLatestPypi },
  ruby:       { label: 'Ruby',       phase0: 'filename:Gemfile.lock',      resolve: resolveLatestRubyGems },
  coldfusion: { label: 'ColdFusion', phase0: 'filename:box.json',          resolve: resolveLatestForgeBox },
  composer:   { label: 'Composer',   phase0: 'filename:composer.json',     resolve: resolveLatestPackagist },
  dart:       { label: 'Dart',       phase0: 'filename:pubspec.yaml',      resolve: resolveLatestPubDev },
  gradle:     { label: 'Gradle',     phase0: 'filename:build.gradle',      resolve: resolveLatestMaven },
};
```

**Phase 0** loops over registry entries to build code search queries.

**Phase 2** uses `ECOSYSTEMS[ecosystem]?.resolve` for version resolution.

Adding a new ecosystem in the future = one new entry in `ECOSYSTEMS` + two new functions (`extract*` + `resolve*`).

---

## Implementation Order

1. **Tab navigation** (Part A) — HTML/CSS restructure + `switchTab()` + remove health from `setPivotView()`
2. **Ecosystem registry** (Part C) — refactor existing 5 ecosystems into registry first
3. **4 missing ecosystems** (Part B) — add as new entries to registry + implement their extract/resolve functions

---

## Files Modified

| File | Changes |
|---|---|
| `index.html` | Tab nav HTML, 4 tab panels, CSS for tab nav, `switchTab()`, `ECOSYSTEMS` registry, 4 new ecosystem extract/resolve functions |

---

## Traceability

| # | Task | Status |
|---|---|---|
| A1 | Add `.tab-nav` HTML with 4 buttons (Scan, Overview, Details, Health) | ✅ done |
| A2 | Wrap config card in `#tab-scan` | ✅ done |
| A3 | Move metrics + effort banner + charts into `#tab-overview` | ✅ done |
| A4 | Move table card (By Repo / By Component only) into `#tab-details` | ✅ done |
| A5 | Create `#tab-health` panel with dedicated `#healthBody` table | ✅ done |
| A6 | Add `.tab-nav`, `.tab-btn`, `.tab-panel` CSS (~60 lines) | ✅ done |
| A7 | Implement `switchTab(tabId)` JS function | ✅ done |
| A8 | Wire tab button click handlers | ✅ done |
| A9 | Auto-switch to Overview after scan completes (`renderDashboard`) | ✅ done |
| A10 | Remove Health case from `setPivotView()`; call `renderHealthTable()` on tab switch | ✅ done |
| A11 | CVE badge on Health tab button (count bubble) | ✅ done |
| A12 | Remove "Health Summary" from `.view-toggle` buttons (now 2 buttons only) | ✅ done |
| A13 | Update chart onClick to switch to Details tab | ✅ done |
| C1 | Define `ECOSYSTEMS` registry object with all 9 ecosystems | ✅ done |
| C2 | Refactor Phase 0 code search to use `ECOSYSTEMS[eco].phase0` | ✅ done |
| C3 | Refactor Phase 2 version resolution to use `ECOSYSTEMS[eco].resolve` | ✅ done |
| B1 | Add `extractColdFusionDeps()` for `box.json` | ✅ done |
| B2 | Add `resolveLatestForgeBox()` via ForgeBox API | ✅ done |
| B3 | Add `extractComposerDeps()` for `composer.json` | ✅ done |
| B4 | Add `resolveLatestPackagist()` via Packagist API | ✅ done |
| B5 | Add `extractDartDeps()` for `pubspec.yaml` | ✅ done |
| B6 | Add `resolveLatestPubDev()` via pub.dev API | ✅ done |
| B7 | Add `extractGradleDeps()` for `build.gradle` / `build.gradle.kts` | ✅ done |
| B8 | Register all 4 new ecosystems in `ECOSYSTEMS` + add checkboxes + filter options | ✅ done |

**Overall: 26 / 26 complete (100%)**

---

## Verification Checklist

1. Page loads → "Scan" tab is active; config card is visible
2. Enter PAT → Load Repos → Scan → scan completes → auto-navigates to "Overview" tab showing metrics + charts
3. Click "Details" tab → dependency table renders with "By Repo" | "By Component" toggle (no Health toggle)
4. Click "Health" tab → Health Summary table renders (one row per scanned repo)
5. If CVEs found → Health tab button shows a red badge with CVE count
6. Enable ColdFusion ecosystem → scan repo with `box.json` → ColdFusion deps appear in Details table
7. Enable Composer ecosystem → scan PHP repo → Composer deps appear
8. Enable Dart ecosystem → scan Flutter repo → pubspec.yaml deps appear
9. Enable Gradle ecosystem → Gradle deps appear, resolved via Maven Central
10. Clicking chart segment → switches to Details tab with filter applied

---

## Commit Reference

`ab30796` on branch `feature/feature-recommendations`
