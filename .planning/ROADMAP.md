# Roadmap: HoochSnapshot

**Project:** HoochSnapshot
**Core Value:** Open the file and immediately know current river conditions at each gauge — no setup, no login, no dependencies.
**Mode:** mvp
**Granularity:** standard
**Created:** 2026-05-17

---

## Phases

- [x] **Phase 1: Static Shell** - Single HTML file opens in a browser with full layout structure, dam section placeholder at top, and station card skeletons for all 9 stations
- [ ] **Phase 2: Live Dashboard** - USGS data fetched and displayed on all station cards, dam iframe embedded with fallback link, station cards navigate to USGS monitoring pages on click

---

## Phase Details

### Phase 1: Static Shell
**Goal:** The HTML file opens directly in any browser with the correct page structure — dam section at top, nine station cards below in the correct order, no external dependencies.
**Mode:** mvp
**Depends on:** Nothing (first phase)
**Requirements:** FILE-01, FILE-02, LAYOUT-01, LAYOUT-02, LAYOUT-03
**Success Criteria** (what must be TRUE):
  1. Opening the `.html` file locally in a browser shows a page without any network requests required to render the layout
  2. A Buford Dam section is visible at the top of the page
  3. Nine station card placeholders appear below the dam section in the exact required order (023344030 first, 02338500 last)
  4. The page has no broken imports, missing stylesheets, or JavaScript errors on open
**Plans:** 1 plan
Plans:
- [x] 01-01-PLAN.md — Write hooch-snapshot.html complete static shell (dam section + nine station cards, all CSS inline, zero external dependencies)

### Phase 2: Live Dashboard
**Goal:** Opening the file fetches live USGS river conditions for all 9 stations and displays them on the cards, the dam schedule is embedded via iframe (with a direct link fallback), and each card is clickable to the USGS monitoring page.
**Mode:** mvp
**Depends on:** Phase 1
**Requirements:** DATA-01, DATA-02, DATA-03, DATA-04, DATA-05, DATA-06, CARD-01, CARD-02, CARD-03, DAM-01, DAM-02
**Success Criteria** (what must be TRUE):
  1. On page load, all 9 station cards populate with live gage height (ft), discharge (cfs), and temperature (°F) values from the USGS API
  2. Stations 02335000, 02335880, and 02336000 show a turbidity value; all other station cards have no turbidity row at all
  3. Each card displays the resolved station name and station ID from the USGS API response
  4. Clicking any station card opens the correct USGS monitoring-location URL in a new browser tab
  5. The Buford Dam section shows either an embedded USACE iframe or a visible direct link if the iframe is blocked by X-Frame-Options
**Plans:** TBD
**UI hint:** yes

---

## Progress

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Static Shell | 1/1 | Complete | 2026-05-17 |
| 2. Live Dashboard | 0/0 | Not started | - |

---

## Coverage

**v1 requirements:** 16 total
**Mapped:** 16/16 ✓

| Requirement | Phase |
|-------------|-------|
| FILE-01 | Phase 1 |
| FILE-02 | Phase 1 |
| LAYOUT-01 | Phase 1 |
| LAYOUT-02 | Phase 1 |
| LAYOUT-03 | Phase 1 |
| DATA-01 | Phase 2 |
| DATA-02 | Phase 2 |
| DATA-03 | Phase 2 |
| DATA-04 | Phase 2 |
| DATA-05 | Phase 2 |
| DATA-06 | Phase 2 |
| CARD-01 | Phase 2 |
| CARD-02 | Phase 2 |
| CARD-03 | Phase 2 |
| DAM-01 | Phase 2 |
| DAM-02 | Phase 2 |

---
*Roadmap created: 2026-05-17*
*Updated: 2026-05-17 — Phase 1 planning complete (1 plan)*
