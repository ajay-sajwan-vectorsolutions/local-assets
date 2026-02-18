# Vector Web Components Dashboard

## What This Is
A single-file (`index.html`) GitHub Pages dashboard that audits `@vector-web-components` usage across repos in the VectorLearning GitHub org. Built with pure HTML/CSS/JS + Chart.js (CDN).

## File Structure
- `index.html` — the entire app (CSS + HTML + JS embedded, ~1500 lines)
- `org_holidays.json` — unrelated holiday data
- `CLAUDE.md` — this file

## Architecture
- **No build tools** — everything lives in `index.html`
- **GitHub API** for repo listing, package.json reading, code search, version resolution
- **npm registry** as fallback for version resolution (private packages likely fail)
- **Chart.js** via CDN for doughnut + stacked bar charts
- **PAT** entered at runtime, never stored

## Key Sections in index.html
- **CSS** (lines ~8–410): All styles including repo picker, pagination, repo group headers
- **HTML** (lines ~410–555): Config card, metrics, charts, results table with pagination
- **JS** (lines ~557–end): State, helpers, repo picker, GitHub API, scan logic, rendering

## Conventions
- Org name `VectorLearning` is hardcoded (display only, not editable)
- Lib repo `VectorLearning/vector-web-components` is hardcoded (display only, not editable)
- Default repos pre-selected: `ts-react`, `eval-pd-spa`, `lti-react`
- Commits go directly to `origin/main`
- `.claude/` is in `.gitignore`

## Scan Flow
1. **Phase 0**: GitHub Code Search pre-filters selected repos for `@vector-web-components` in package.json
2. **Phase 1**: Scan matching repos concurrently (batches of 5) for component dependencies
3. **Phase 2**: Resolve latest versions from lib repo (tags, releases, packages/*/package.json), npm fallback

## Important Implementation Details
- **AbortController** threads through all fetch calls for scan cancellation
- `.card` CSS must NOT have `overflow: hidden` — it clips the repo picker dropdown
- `.config-card` uses `overflow: visible`; `.table-card` uses `overflow: hidden`
- Results table is grouped by repo with header rows; sorted by repo then severity within repo
- `filteredResults` is the active dataset for pagination and detail row expansion
