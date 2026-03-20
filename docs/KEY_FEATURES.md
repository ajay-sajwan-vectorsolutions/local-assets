# Key Features — Vector Web Components Dashboard

## Overview

The **Vector Web Components Dashboard** is a single-page, browser-based tool hosted on GitHub Pages. It lets developers audit which `@vector-web-components/*` packages are installed across repos in the **VectorLearning** GitHub organization, which versions those repos are on, and which upgrades are available.

No build tools, no backend — everything runs in the browser using the GitHub API, the npm registry (fallback), and Chart.js for visualization.

---

## 1. Configuration Panel

The configuration card sits at the top of the page and collects all inputs needed before a scan can run.

| Field | Behavior |
|---|---|
| **GitHub Organization** | Read-only label — always shows `VectorLearning`. |
| **GitHub Personal Access Token** | Password input. Required for every GitHub API call. Never stored outside the browser tab. |
| **Repos to Scan** | Multi-select repo picker (see §3). |
| **Component Library Repo** | Read-only label — always shows `VectorLearning/vector-web-components`. Used as the authoritative source for latest versions. |

### Config lockdown during scan
When a scan is in progress the entire config card body is visually dimmed (`opacity: 0.55`) and all inputs are blocked (`pointer-events: none`). The action buttons (Scan / Cancel / Clear) remain fully interactive. This prevents accidental changes mid-scan.

---

## 2. PAT Guide — In-Browser Wizard

A **"How to generate"** link next to the PAT field opens a 7-step modal wizard that walks the user through creating a GitHub Personal Access Token without leaving the dashboard.

### Steps
| # | Title | Key instruction |
|---|---|---|
| 1 | Open GitHub Settings | Profile picture → Settings |
| 2 | Go to Developer Settings | Bottom of the left sidebar |
| 3 | Select Tokens (classic) | Personal access tokens → Tokens (classic) |
| 4 | Generate New Token (classic) | Dropdown → Generate new token (classic) |
| 5 | Configure the Token | Name it, set expiration, check the **repo** scope |
| 6 | Enable read:org Scope | Check **read:org** under admin:org |
| 7 | Generate & Copy the Token | Click Generate token — copy immediately |

Each step shows a screenshot (`docs/1.png` … `docs/6.png`) and a plain-English description. Steps 5 and 6 include a highlighted tip box explaining _why_ each scope is needed.

### Navigation
- **Prev / Next** buttons step through slides
- **Dot indicators** at the bottom show current position; clicking a dot jumps directly to that step
- The **Next** button becomes **Done** on the last step and closes the modal
- Pressing **Escape** or clicking the dark overlay also closes the modal

---

## 3. Repo Picker

The repo picker is a custom multi-select dropdown that replaces a plain `<select>` element.

### Load Repos button
Before the picker is usable, the user clicks **Load Repos**. This fires two parallel requests:
- Paginates through the GitHub org's repos (`GET /orgs/:org/repos`) and sorts them alphabetically.
- Fetches custom org-level properties (`Product`, `Repo-Class`) for later use as filters.

After loading, three repos are **pre-selected by default**: `ts-react`, `eval-pd-spa`, `lti-react`.

### Dropdown behavior
- Click the trigger field to open the dropdown (only works after repos are loaded).
- A **search box** at the top filters the list in real time by repo name.
- **Select All / Deselect All** toggles all currently-visible repos (respects the active search filter).
- Each row has a checkbox and label. Clicking anywhere on the row toggles selection.

### Chip tags
- When 1–5 repos are selected, the trigger field shows a **chip tag** for each repo name with a `×` button to deselect individually.
- When more than 5 repos are selected, the trigger shows a count summary (e.g. `12 repos selected`).
- Clicking outside the dropdown closes it automatically.

---

## 4. Scan Controls

Three buttons sit below the config fields:

| Button | Description |
|---|---|
| **Scan Repos** | Validates inputs, then starts a three-phase scan. Disabled (grayed out) while scanning. |
| **Cancel** | Visible only during a scan. Calls `AbortController.abort()` to immediately cancel all in-flight requests. |
| **Clear** | Hides the results section and resets the status message. |

A **status message** to the right of the buttons reports progress and errors inline (e.g. "Scanning repos 1–5 of 12…").

### Progress bar
A slim blue-to-purple gradient bar below the buttons animates from 0 → 100% across the three scan phases:
- 0–10%: Phase 0 (code search pre-filter)
- 10–40%: Phase 1 (repo scanning)
- 40–95%: Phase 2 (version resolution)
- 100%: Scan complete (bar hides)

---

## 5. Metrics Row

After a successful scan, four metric cards appear:

| Card | Color | Metric |
|---|---|---|
| **Repos Scanned** | Neutral | Number of repos that were actually scanned (after pre-filtering) |
| **Unique Components** | Neutral | Count of distinct `@vector-web-components/*` package names found |
| **Up to Date** | Green | Installed versions that match the latest release |
| **Upgrades Available** | Amber | Installed versions that are behind the latest (patch, minor, or major) |

If any components could not have their version resolved, the "Upgrades Available" detail text appends `(N unknown)`.

---

## 6. Charts

Two Chart.js charts appear side-by-side in a two-column grid (stacks to one column on narrow screens).

### Components per Repository — Doughnut chart
- One slice per repo, sized by number of `@vector-web-components` usages found in that repo.
- Legend positioned on the right.
- Tooltip shows: `<repo-name>: N components (X%)`.

### Version Freshness — Horizontal stacked bar chart
- One bar per unique component (short name, e.g. `button`).
- Bar segments are color-coded by status:
  - **Green** — Up to date
  - **Blue** — Patch behind
  - **Amber** — Minor update available
  - **Red** — Major update available
  - **Gray** — Version unknown
- Each segment's value is the number of repos using that component at that staleness level.

---

## 7. Results Table

A full-width table card below the charts shows every component usage found.

### Columns
| Column | Content |
|---|---|
| **File** | Path to the `package.json` where the dependency was found (e.g. `apps/web/package.json`) |
| **Package Name** | Short component name (e.g. `button` — `@vector-web-components/` prefix stripped) |
| **Installed** | Installed version badge (gray) |
| **Latest** | Latest known version badge (green), or `N/A` if unresolvable |
| **Status** | Severity badge — Up to date / Patch available / Minor update / Major update / Unknown |
| **Type** | Dependency type — `dependencies`, `devDependencies`, or `peerDependencies` |
| **Details** | "View changes" button (only shown when an upgrade is available) |

### Grouped by repo
Results are grouped with a **gray header row** per repo showing the repo name and component count. Within each repo group, rows are sorted by severity (major → minor → patch → unknown → current), then alphabetically by package name.

### Toolbar filters
Above the table, three filter controls narrow the displayed rows in real time:
- **Text search** — matches against repo name, package name, file path, version, and dependency type.
- **Product** dropdown — filters by the `Product` custom org property (populated from GitHub org properties).
- **Repo Class** dropdown — filters by the `Repo-Class` custom org property.

All three filters apply simultaneously (`AND` logic). The page resets to page 1 on any filter change.

### Pagination
When there are more than 25 result rows, a pagination toolbar appears at the bottom of the table:
- **Row count** info (e.g. `Showing 1–25 of 87 results`)
- **Rows per page** selector: 25 / 50 / 100
- **Page buttons** with Prev / Next; up to 7 numbered buttons shown with ellipsis for large page counts
- Changing page smoothly scrolls the table card into view

---

## 8. "View Changes" Detail Panel

For any component that is not on the latest version, the **View changes** button expands an inline detail panel directly below that table row.

The panel contains:
- A header line: `<Upgrade type>: <current version> → <latest version> (N versions behind)`
- A **version timeline** — a vertical list of every intermediate release between the installed version and the latest, showing:
  - Version number (amber dot for installed, green dot for latest)
  - Release date (if available)
  - Up to 10 bullet points of release notes from the GitHub release body
  - If more than 10 intermediate versions exist, a `…and N more versions` note is shown
- If no version history could be retrieved (e.g. private npm package), a fallback message suggests entering the component library's GitHub repo path.

Clicking **View changes** again (or **Hide**) collapses the panel.

---

## 9. Responsive Layout

The dashboard adapts to narrower viewports at ≤ 900px:
- Container padding reduces from `40px` to `16px`.
- The config form grid collapses from two columns to one.
- The metrics row collapses from four columns to two.
- The charts row collapses from two columns to one.
- The PAT guide modal expands to 95% of the viewport width.
