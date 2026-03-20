# Feature Recommendations — Vector Web Components Dashboard

> Generated: 2026-03-20
> Based on full code analysis of `index.html`

## Current Gaps

| Gap | Impact |
|---|---|
| PAT and repo selection are lost on page refresh | Friction on every session |
| No severity/status filter in the table toolbar | Can't isolate "show me only major upgrades" |
| No direct link to the file in GitHub | Developer has to hunt for it manually |
| No copy-to-clipboard for the upgrade command | Manual lookup of `npm install @pkg@version` |
| Charts are decorative — clicking a segment doesn't filter the table | Missed interactivity opportunity |
| No component-pivot view (all repos using component X) | Hard to assess blast radius of a component change |
| No export (CSV/JSON) | Can't share results with non-technical stakeholders |
| No scan history / diff | Can't tell if things got better or worse since last week |
| No total upgrade-effort summary | Leadership can't see org-wide health at a glance |
| Deprecated packages not detected | Team wouldn't know a package was sunset |

---

## Tier 1 — Quick Wins (high value, low effort)

### 1. Severity Filter in Table Toolbar

Add a **Status** `<select>` to the existing toolbar filter row:

```
All Statuses | Major update | Minor update | Patch available | Up to date | Unknown
```

- Filters `filteredResults` the same way Product/Repo-Class filters do (AND logic)
- Lets a lead immediately isolate "what's blocking us from being current"
- **File:** `index.html` — `filterTable()`, toolbar HTML, `populateResultFilters()`

---

### 2. Direct GitHub File Links

In each table row, the **File** cell currently shows plain text (e.g. `apps/web/package.json`). Make it a hyperlink:

```
https://github.com/VectorLearning/<repo>/blob/main/<file>
```

One click from audit finding to the actual dependency declaration.

- **File:** `index.html` — `renderTable()`, the `<td>` for `r.file`

---

### 3. Copy Upgrade Command

Add a copy icon next to the **Latest** version badge for any row that has an upgrade available. Clicking copies:

```
npm install @vector-web-components/<name>@<latestVersion>
```

Eliminates a manual lookup step for every developer acting on results.

- **File:** `index.html` — `renderTable()`, `statusBadge()` area

---

### 4. LocalStorage Persistence for PAT and Selected Repos

- Save the PAT value to `sessionStorage` (clears on tab close — avoids long-term token storage risk)
- Save selected repo names to `localStorage` (persists across refreshes)
- Restore both on page load before the user has to re-enter anything

**Org value:** Eliminates repeated re-entry for power users who run the dashboard daily.

- **File:** `index.html` — `loadOrgRepos()`, `toggleRepoSelection()`, `ghToken` input event

---

### 5. Upgrade Effort Summary Banner

A compact summary bar above the results table (or below the metric cards):

```
Org upgrade backlog:  🔴 4 major  🟡 11 minor  🔵 6 patch  across 8 repos
```

Gives leadership a single-line health status without reading the full table.

- **File:** `index.html` — `renderDashboard()`, new HTML element

---

## Tier 2 — Medium Effort (high org value)

### 6. Chart → Table Drill-down

Clicking a doughnut slice (repo) or a bar segment (component + severity) auto-applies the matching filters to the table and scrolls to it. Use Chart.js `onClick` callback → call `filterTable()` with preset values.

**Org value:** Makes charts actionable, not decorative. Demo-friendly for leads.

- **File:** `index.html` — chart initialization, `filterTable()`

---

### 7. Component-Pivot View

Toggle button: **"By Repo"** (current) vs **"By Component"**. In component view, the table groups by package name instead of repo, showing all repos pinned at each version. Makes it easy to answer "how many teams are still on v1 of `<button>`?"

**Org value:** Critical for the component library team to prioritize releases.

- **File:** `index.html` — `renderTable()`, new toggle control in table card header

---

### 8. Export to CSV / JSON

A download button in the table card header exports `filteredResults` as:

- **CSV** — one row per result, all columns including Product and Repo-Class
- **JSON** — full structured dump for import into other tools (Grafana, Confluence)

**Org value:** Non-engineers (PMs, architects) can paste CSV into spreadsheets for planning.

- **File:** `index.html` — table card header, new export helper functions

---

### 9. Scan History via LocalStorage

After each scan, serialize `allResults` + timestamp into `localStorage` (keyed by date). On next scan, diff against the previous results:

- Highlight rows that **regressed** (version went from current → outdated — e.g. new component added at old version)
- Show a "since last scan" delta on each metric card (e.g. `+2 major`)

**Org value:** Demonstrates progress (or regression) over time without any backend.

- **File:** `index.html` — post-scan logic in `runScan()`, metric card rendering

---

## Tier 3 — Strategic / Org-Level

### 10. Jira Ticket Creation (via Atlassian MCP)

A **"Create Jira ticket"** action per table row (or a **"Bulk create tickets"** for all major upgrades). Pre-fills:

- **Summary:** `Upgrade @vector-web-components/<name> in <repo>: <current> → <latest>`
- **Description:** version timeline + release notes from the detail panel
- **Labels:** `vwc-upgrade`, severity

**Org value:** Closes the loop from audit → tracked work item without copy-paste. The Atlassian MCP server is available in Claude Code sessions for implementation.

- **File:** `index.html` — row action buttons in `renderTable()`

---

### 11. Deprecated / End-of-Life Package Detection

During Phase 2 version resolution, check the npm registry response for the `deprecated` field on the installed version. Surface a `⚠ Deprecated` badge in the Status column.

**Org value:** Prevents teams from staying on a package version that's been sunset by the library team.

- **File:** `index.html` — Phase 2 version resolution logic, `statusBadge()`

---

### 12. Shareable Scan URL (URL Hash State)

Encode `selectedRepos` + active filters into the URL hash (`#repos=ts-react,lti-react&product=RedVector+LMS`). Anyone opening that URL sees the same filtered view. No PAT encoded (security).

**Org value:** Lets a lead share a pre-filtered view ("all major upgrades in RedVector LMS repos") in Slack or email without explaining how to set the filters.

- **File:** `index.html` — filter change handlers, page load init

---

### 13. Scheduled Auto-Refresh

A **"Auto-refresh every N minutes"** toggle (using `setInterval`). Useful when the dashboard is displayed on a team TV/monitor. Re-runs the last scan config without manual intervention.

- **File:** `index.html` — config card, `runScan()` entry point

---

### 14. Per-Repo Upgrade Readiness Score

Compute a score per repo: `(up-to-date / total) * 100%`. Add a readiness column or sort option. Color-code repos in the group headers (green ≥ 80%, amber 50–79%, red < 50%).

**Org value:** Instant portfolio health view. Useful for quarterly planning and release readiness reviews.

- **File:** `index.html` — repo group header rendering in `renderTable()`

---

## Recommended Implementation Order

| Priority | Feature | Effort |
|---|---|---|
| 1 | Severity filter in toolbar | 1–2 hrs |
| 2 | Direct GitHub file links | 30 min |
| 3 | Copy upgrade command | 1 hr |
| 4 | LocalStorage PAT/repo persistence | 1–2 hrs |
| 5 | Upgrade effort summary banner | 1 hr |
| 6 | Chart → table drill-down | 2–3 hrs |
| 7 | Export to CSV/JSON | 2 hrs |
| 8 | Component-pivot view | 3–4 hrs |
| 9 | Scan history / diff | 3–4 hrs |
| 10 | Jira ticket creation | 3–5 hrs |

All changes are self-contained within `index.html` — no build system required.
