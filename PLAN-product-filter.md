# Plan: Property Filters (Product + Repo-Class) Using GitHub Custom Properties

## Project Context
- **Repo**: `ajay-sajwan-vectorsolutions/local-assets` — hosted on GitHub Pages
- **App**: Single-file HTML dashboard (`index.html`) that audits `@vector-web-components` usage across the **VectorLearning** GitHub org
- **Stack**: Pure HTML/CSS/JS + Chart.js (CDN). No build tools or frameworks.
- **Current state**: Working dashboard with multi-select repo picker, scan optimizations, grouped results table with pagination, and sticky headers

## Problem
The dashboard currently shows all org repos in a flat list. Users want to filter repos by **Product** (e.g., "RedVector LMS", "SafeLMS") and **Repo-Class** (e.g., "Product Source Code", "Supporting Code - IaC") before selecting which repos to scan. These classifications already exist as **GitHub Custom Properties** on the VectorLearning org.

## Data Source: GitHub Organization Custom Properties

### What Already Exists
The VectorLearning org has custom properties defined. We use two:

| Property | Type | Description |
|---|---|---|
| **Product** | `single_select` (38 values) | Product/LMS this repo belongs to |
| **Repo-Class** | `single_select` (7 values) | Type of code (Product Source Code, IaC, Tooling, etc.) |

### API Endpoints

| Endpoint | Purpose | Pages | Time | Payload |
|---|---|---|---|---|
| `GET /orgs/{org}/properties/schema` | Get property definitions + allowed values | 1 | ~0.5s | ~3 KB |
| `GET /orgs/{org}/properties/values?per_page=100` | Get all repos with property values | 3 | ~1.5s | ~110 KB |

### Performance
- **256 repos** across 3 pages — total ~1.5s, ~110 KB
- Runs in parallel with existing repo listing — **zero added wait time**

---

## UI Layout: Option A (Filter Row Above Repo Picker)

Two `<select>` dropdowns in a row between the PAT/Org fields and the Repos-to-Scan picker. Both dropdowns are **disabled** until "Load Repos" completes.

```
┌─────────────────────────────────────────────────────────────────┐
│  Configuration                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  GitHub Organization          GitHub Personal Access Token      │
│  ┌─────────────────────┐      ┌──────────────────────────┐     │
│  │ VectorLearning      │      │ ghp_xxxxxxxxxxxx         │     │
│  └─────────────────────┘      └──────────────────────────┘     │
│                                                                 │
│  Product                        Repo Class                      │
│  ┌──────────────────────────┐   ┌──────────────────────────┐   │
│  │ All Products          ▼  │   │ All Classes           ▼  │   │
│  └──────────────────────────┘   └──────────────────────────┘   │
│  (disabled until repos loaded — grayed out with cursor:no-drop) │
│                                                                 │
│  Repos to Scan                  Component Library Repo          │
│  ┌─────────────────────┐        ┌──────────────────────────┐   │
│  │ 5 repos selected  ▼ │ [Load] │ VectorLearning/vector..  │   │
│  └─────────────────────┘        └──────────────────────────┘   │
│                                                                 │
│  [ Scan Repos ]  [ Cancel ]  [ Clear ]                          │
└─────────────────────────────────────────────────────────────────┘
```

After "Load Repos" populates, the dropdowns become enabled and show counts:
```
  Product                           Repo Class
  ┌────────────────────────────┐    ┌──────────────────────────────┐
  │ All Products (256)      ▼  │    │ All Classes (256)         ▼  │
  ├────────────────────────────┤    ├──────────────────────────────┤
  │ RedVector LMS (12)         │    │ Product Source Code (142)    │
  │ SafeLMS (8)                │    │ Supporting Code - IaC (28)   │
  │ Convergence LMS (15)       │    │ Supporting Code - CI/CD (19) │
  │ Target Solutions LMS (9)   │    │ ...                          │
  │ ...                        │    └──────────────────────────────┘
  │ Unassigned (3)             │
  └────────────────────────────┘
```

---

## Cross-Filter Logic

The two dropdowns and the repo picker form a **cascading filter chain**. Changing any filter updates the others.

### Filter Chain

```
Product dropdown ──┐
                   ├──→ Filtered repo set ──→ Repo picker list
Repo-Class dropdown┘
```

### How Cross-Filtering Works

Each dropdown option shows a **count** of repos that match the **combined** active filters. When the user changes one dropdown, the other dropdown's counts update to reflect what's possible given the current selection.

#### Example Walkthrough

**Initial state** (no filters):
```
Product:    All Products (256)     ← shows total repos
Repo-Class: All Classes (256)      ← shows total repos
Repo picker: 256 repos visible
```

**User selects Product = "RedVector LMS"**:
```
Product:    RedVector LMS (12)     ← 12 repos are RedVector
Repo-Class: All Classes (12)       ← now shows 12 (scoped to RedVector)
  - Product Source Code (8)         ← 8 of the 12 RedVector repos are source code
  - Supporting Code - IaC (2)       ← 2 are IaC
  - Supporting Code - CI/CD (1)     ← 1 is CI/CD
  - Documentation (1)               ← 1 is docs
  - Supporting Code - QA (0)        ← 0 match — still shown but grayed/disabled
Repo picker: 12 repos visible (all RedVector repos)
```

**User then selects Repo-Class = "Product Source Code"**:
```
Product:    RedVector LMS (8)      ← now 8 (RedVector + Source Code)
Repo-Class: Product Source Code (8) ← 8 repos match both filters
Repo picker: 8 repos visible
```

**User resets Product to "All Products"**:
```
Product:    All Products (142)     ← 142 repos are Source Code across all products
Repo-Class: Product Source Code (142)
Repo picker: 142 repos visible (all Source Code repos)
```

### Implementation: `applyPropertyFilters()`

This is the core function called whenever either dropdown changes.

```
function applyPropertyFilters():
  1. Read selectedProduct from #filterProduct.value
  2. Read selectedRepoClass from #filterRepoClass.value

  3. Compute filteredRepoSet:
     - Start with all orgRepos
     - If selectedProduct != '' → keep only repos where productMap[repo] === selectedProduct
     - If selectedRepoClass != '' → keep only repos where repoClassMap[repo] === selectedRepoClass
     - Result: filteredRepoSet (array of repo names matching ALL active filters)

  4. Update Product dropdown counts:
     - For each Product value, count how many repos in orgRepos match:
       - that Product value AND the currently selected Repo-Class (if any)
     - Update option text: "RedVector LMS (12)" → "RedVector LMS (8)"
     - Disable options with count 0 (grayed out, not removed)

  5. Update Repo-Class dropdown counts:
     - For each Repo-Class value, count how many repos in orgRepos match:
       - that Repo-Class value AND the currently selected Product (if any)
     - Update option text similarly
     - Disable options with count 0

  6. Update the "All" option counts:
     - "All Products (N)" where N = repos matching current Repo-Class filter (or total)
     - "All Classes (N)" where N = repos matching current Product filter (or total)

  7. Update repo picker:
     - Call renderRepoPickerList() which now respects the filtered set
     - The repo picker only shows repos in filteredRepoSet

  8. Handle selected repos that fall outside the new filter:
     - Deselect any repos in selectedRepos that are NOT in filteredRepoSet
     - Update the trigger display via updateRepoPickerTrigger()
     - Show status message if repos were deselected: "N repos deselected (filtered out)"
```

### Affected Existing Functions

| Function | Change |
|---|---|
| `renderRepoPickerList(filter)` | Add property filter: only render repos that are in `getFilteredRepos()`. The search `filter` param still applies on top. |
| `toggleSelectAll()` | Only select/deselect repos visible after **both** property filters and search filter. |
| `updateSelectAllButton()` | Same — scope "all" to the currently visible (filtered) set. |
| `filterRepos()` | No change needed — it calls `renderRepoPickerList(filter)` which already applies property filters. |

### New Helper: `getFilteredRepos()`

Returns the subset of `orgRepos` matching both active property filters:

```js
function getFilteredRepos() {
  return orgRepos.filter(repo => {
    if (selectedProduct && productMap[repo] !== selectedProduct) return false;
    if (selectedRepoClass && repoClassMap[repo] !== selectedRepoClass) return false;
    return true;
  });
}
```

This is used by `renderRepoPickerList()`, `toggleSelectAll()`, and `applyPropertyFilters()`.

---

## State Variables

```js
// Property filter state
let productMap = {};         // { repoName: "RedVector LMS", ... }
let repoClassMap = {};       // { repoName: "Product Source Code", ... }
let productValues = [];      // ["Acadis", "Casino Essentials LMS", ...] sorted
let repoClassValues = [];    // ["Documentation", "Product Source Code", ...] sorted
let selectedProduct = '';    // Current product filter (empty = all)
let selectedRepoClass = '';  // Current repo-class filter (empty = all)
```

---

## Updated `loadOrgRepos()` Flow

```
loadOrgRepos():
  1. Disable Load Repos button, show spinner
  2. Promise.all([
       paginateOrgRepos(org, token),       // existing — returns repo names
       fetchPropertySchema(org, token),    // new — returns { productValues, repoClassValues }
       fetchPropertyValues(org, token),    // new — returns { productMap, repoClassMap }
     ])
  3. On success:
     a. Store orgRepos, productMap, repoClassMap, productValues, repoClassValues
     b. Populate Product dropdown with sorted productValues + counts
     c. Populate Repo-Class dropdown with sorted repoClassValues + counts
     d. Enable both dropdowns (remove disabled attribute)
     e. Pre-select default repos
     f. Render repo picker list
  4. On custom-properties failure (graceful fallback):
     a. Leave dropdowns disabled
     b. Repo picker still works — all repos shown, no filtering
     c. Log warning to console
```

---

## `fetchPropertySchema()` Function

```
async function fetchPropertySchema(org, token, signal):
  1. Call GET /orgs/{org}/properties/schema
  2. Find the "Product" property → extract allowed_values → sort alphabetically → store as productValues
  3. Find the "Repo-Class" property → extract allowed_values → sort alphabetically → store as repoClassValues
  4. Return { productValues, repoClassValues }
  5. If API fails → return empty arrays (graceful fallback)
```

## `fetchPropertyValues()` Function

```
async function fetchPropertyValues(org, token, signal):
  1. Paginate GET /orgs/{org}/properties/values?per_page=100 (3 pages for 256 repos)
  2. For each repo in the response:
     - Extract "Product" property value → productMap[repo_name] = value || "Unassigned"
     - Extract "Repo-Class" property value → repoClassMap[repo_name] = value || "Unassigned"
  3. Return { productMap, repoClassMap }
  4. If API fails → return empty maps (graceful fallback)
```

---

## Populating Dropdowns: `populateFilterDropdowns()`

Called once after `loadOrgRepos()` succeeds. Builds the `<option>` elements with counts.

```
function populateFilterDropdowns():
  1. Product dropdown (#filterProduct):
     - Clear existing options
     - Add "All Products (N)" where N = orgRepos.length
     - For each value in productValues:
       - Count repos where productMap[repo] === value
       - Add option: "ValueName (count)"
     - If any repos have "Unassigned" → add "Unassigned (count)" at the end
     - Enable the dropdown (remove disabled)

  2. Repo-Class dropdown (#filterRepoClass):
     - Same logic using repoClassValues and repoClassMap
     - Enable the dropdown (remove disabled)
```

---

## CSS Changes (~20 lines)

```css
/* Property Filter Row */
.filter-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.filter-row select {
  padding: 10px 14px;
  border: 1px solid var(--border);
  border-radius: 8px;
  font-size: 0.9rem;
  background: var(--surface-alt);
  color: var(--text);
  cursor: pointer;
  transition: all 0.15s ease;
  width: 100%;
}

.filter-row select:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.15);
  background: #fff;
}

.filter-row select:disabled {
  color: var(--text-muted);
  cursor: not-allowed;
  opacity: 0.6;
}

/* Responsive */
@media (max-width: 900px) {
  .filter-row { grid-template-columns: 1fr; }
}
```

## HTML Changes (~15 lines)

Insert inside `.form-grid`, after the PAT form-group and before the "Repos to Scan" form-group:

```html
<!-- Property Filters -->
<div class="form-full">
  <div class="filter-row" id="propertyFilters">
    <div class="form-group">
      <label>Product</label>
      <select id="filterProduct" onchange="applyPropertyFilters()" disabled>
        <option value="">All Products</option>
      </select>
    </div>
    <div class="form-group">
      <label>Repo Class</label>
      <select id="filterRepoClass" onchange="applyPropertyFilters()" disabled>
        <option value="">All Classes</option>
      </select>
    </div>
  </div>
  <span class="help-text">Filters populate after loading repos. Narrow the repo list by product or class.</span>
</div>
```

## JS Changes Summary (~100 lines)

| Section | Change | Lines (est.) |
|---|---|---|
| State variables | Add `productMap`, `repoClassMap`, `productValues`, `repoClassValues`, `selectedProduct`, `selectedRepoClass` | 6 |
| `fetchPropertySchema(org, token, signal)` | New — fetch schema, extract allowed values for Product and Repo-Class | 15 |
| `fetchPropertyValues(org, token, signal)` | New — paginate values endpoint, build productMap + repoClassMap | 20 |
| `loadOrgRepos()` | Wrap existing fetch in `Promise.all` with schema + values calls; call `populateFilterDropdowns()` on success; enable dropdowns | 15 |
| `populateFilterDropdowns()` | New — build `<option>` elements with counts for both dropdowns | 15 |
| `applyPropertyFilters()` | New — core cross-filter logic: update counts, update repo picker, deselect filtered-out repos | 25 |
| `getFilteredRepos()` | New — returns repos matching both active filters | 5 |
| `renderRepoPickerList(filter)` | Modify — filter against `getFilteredRepos()` before rendering | 3 |
| `toggleSelectAll()` / `updateSelectAllButton()` | Modify — scope to `getFilteredRepos()` | 3 |

**Total estimated: ~100 lines of JS, ~15 lines of HTML, ~20 lines of CSS.**

---

## Error Handling

| Scenario | Behavior |
|---|---|
| Custom properties API fails (403/404) | Log warning. Dropdowns stay disabled. Dashboard works normally without filters. |
| PAT lacks `read:org` scope | Same graceful fallback — dropdowns disabled, repo picker unfiltered. |
| Repo has `null` Product or Repo-Class value | Mapped to `"Unassigned"`. Shown in dropdown as `"Unassigned (N)"`. |
| Schema missing Product or Repo-Class property | That dropdown stays disabled. The other dropdown still works. |
| Values endpoint returns repos not in org listing | Ignored — only repos in `orgRepos` are used for counts and filtering. |

## Edge Cases

| Case | Handling |
|---|---|
| User changes filter after selecting repos | Deselect repos that no longer match. Show status: "N repos deselected (filtered out)". |
| "All" selected on both dropdowns | Show all repos. Default state. |
| Filter combination yields 0 repos | Repo picker shows "No repos match your filters." message. |
| Option has 0 matching repos | Shown in dropdown with `(0)` count but visually dimmed / disabled. |
| AbortController cancellation | Signal threaded through all new fetch calls. |
| Load Repos clicked again (re-fetch) | Reset both dropdowns to "All", clear selectedProduct/selectedRepoClass, re-populate. |

---

## Implementation Order

1. Add new state variables
2. Add CSS for `.filter-row` and select styling
3. Add HTML filter row in `.form-grid`
4. Implement `fetchPropertySchema()` and `fetchPropertyValues()`
5. Update `loadOrgRepos()` to call them in parallel and populate dropdowns
6. Implement `getFilteredRepos()` helper
7. Implement `applyPropertyFilters()` with cross-filter count updates
8. Update `renderRepoPickerList()` to use `getFilteredRepos()`
9. Update `toggleSelectAll()` / `updateSelectAllButton()` to respect filters
10. Test with live PAT

## Verification

1. Open `index.html` → confirm both dropdowns are disabled/grayed out
2. Enter PAT → click "Load Repos" → confirm:
   - Both dropdowns become enabled
   - Product shows 38 values + "Unassigned" with correct counts
   - Repo-Class shows 7 values + "Unassigned" with correct counts
3. Select Product = "RedVector LMS" → confirm:
   - Repo picker shows only RedVector repos
   - Repo-Class counts update to reflect only RedVector repos
   - Options with 0 repos are dimmed
4. Select Repo-Class = "Product Source Code" → confirm:
   - Repo picker narrows further
   - Product counts update to reflect Source Code filter
5. Reset Product to "All Products" → confirm Repo-Class counts update
6. Reset both to "All" → confirm all repos shown
7. Select a filter, pick repos, change filter → confirm repos outside new filter are deselected
8. Test with PAT lacking `read:org` → confirm dropdowns stay disabled, picker works normally
9. Test cancel during Load Repos → confirm all fetches abort cleanly

## Cleanup

After implementation, delete the mockup files:
- `option-a.html`
- `option-b.html`
