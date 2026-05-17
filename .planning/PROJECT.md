# HoochSnapshot

## What This Is

A single self-contained HTML file that displays live Chattahoochee River conditions — gage height, discharge, water temperature (°F), and turbidity — pulled directly from USGS Water Services on page load. It also embeds the USACE Hydropower schedule for Buford Dam. Opens in any browser, no server required.

## Core Value

Open the file and immediately know current river conditions at each gauge — no setup, no login, no dependencies.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Single `.html` file — zero external dependencies, opens locally in any browser
- [ ] Fetches live data from USGS Instantaneous Values API on page load for 9 stations
- [ ] Stations: 023344030, 02335000, 02335450, 02335815, 02335880, 02336000, 02337170, 02338000, 02338500
- [ ] Each station card shows: gage height, discharge, water temperature (°F), turbidity (where available)
- [ ] Clicking a station card opens the USGS monitoring page for that station in a new tab
- [ ] Embeds USACE Hydropower schedule for Buford Dam (Lake Lanier) via iframe
- [ ] Iframe fallback: if USACE blocks embedding (X-Frame-Options), show a direct link instead
- [ ] Dashboard card layout — clean, readable, easy to scan

### Out of Scope

- Server, backend, or build system — must remain a single static file
- Offline/cached mode — live fetch on open is sufficient
- Multiple users or sharing features — personal use only
- Historical data charts or graphs — current conditions only
- Alerts or notifications — passive display only
- Other rivers or gauge networks — Chattahoochee only

## Context

- **USGS Instantaneous Values API**: `https://waterservices.usgs.gov/nwis/iv/` — public, no auth, JSON response. Parameter codes: 00065 (gage height, ft), 00060 (discharge, cfs), 00010 (water temp, °C → convert to °F), 63680 or 00076 (turbidity, FNU). Not all stations report all parameters.
- **USGS monitoring page URL pattern**: `https://waterdata.usgs.gov/monitoring-location/{station_id}/`
- **USACE Hydropower page**: `https://spatialdata.usace.army.mil/Hydropower/` — JavaScript SPA, no public REST API discovered. Embedding via iframe; government sites sometimes set X-Frame-Options headers that block iframe rendering.
- **Personal use**: no accessibility, i18n, or multi-device requirements beyond "works in a modern browser"

## Constraints

- **Tech stack**: Plain HTML/CSS/JavaScript — no frameworks, no bundlers, no npm. Must remain a single file.
- **Data access**: USGS API is public and CORS-friendly. USACE iframe may be blocked by X-Frame-Options — handle gracefully.
- **Deployment**: None. File lives locally and is opened directly in a browser.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single HTML file | "Transportable" — shareable, no setup, works anywhere | — Pending |
| Fetch live on open | Always current; no stale data problem | — Pending |
| Iframe for USACE data | No public API found; iframe preserves full functionality | — Pending |
| Link fallback for iframe | X-Frame-Options may block embed; degrade gracefully | — Pending |
| Temperature in °F | User preference; USGS reports °C, convert in JS | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-05-17 after initialization*
