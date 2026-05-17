# Requirements: HoochSnapshot

**Defined:** 2026-05-17
**Core Value:** Open the file and immediately know current river conditions at each gauge — no setup, no login, no dependencies.

## v1 Requirements

### File & Delivery

- [ ] **FILE-01**: Entire application is a single `.html` file with no external dependencies or build step
- [ ] **FILE-02**: File opens directly in any modern browser without a local server

### Layout & Navigation

- [ ] **LAYOUT-01**: Buford Dam release schedule section appears at the top of the page
- [ ] **LAYOUT-02**: USGS station cards appear below the dam schedule in this exact order: 023344030, 02335000, 02335450, 02335815, 02335880, 02336000, 02337170, 02338000, 02338500
- [ ] **LAYOUT-03**: Each station is displayed as a card in a dashboard card layout

### Data Fetching

- [ ] **DATA-01**: Page fetches live data from the USGS Instantaneous Values API on page load for all 9 stations
- [ ] **DATA-02**: Fetches gage height (parameter 00065, unit: ft) for each station
- [ ] **DATA-03**: Fetches discharge (parameter 00060, unit: cfs) for each station
- [ ] **DATA-04**: Fetches water temperature (parameter 00010, unit: °C) and displays it converted to °F
- [ ] **DATA-05**: Fetches turbidity (parameter 63680) where available; shows "—" for stations that don't report it
- [ ] **DATA-06**: Stations that don't report a given parameter show "—" rather than an error

### Station Cards

- [ ] **CARD-01**: Each station card displays the station name (resolved from USGS API) and station ID
- [ ] **CARD-02**: Each station card shows gage height, discharge, temperature (°F), and turbidity (or "—")
- [ ] **CARD-03**: Clicking a station card opens `https://waterdata.usgs.gov/monitoring-location/{station_id}/` in a new tab

### Dam Release Schedule

- [ ] **DAM-01**: USACE Hydropower page for Buford Dam (`https://spatialdata.usace.army.mil/Hydropower/`) is embedded via iframe at the top of the page
- [ ] **DAM-02**: If the iframe is blocked by X-Frame-Options headers, a direct link to the USACE page is shown as a fallback

## v2 Requirements

### Enhancements

- **ENH-01**: Manual refresh button to re-fetch all data without reloading the page
- **ENH-02**: Color-coded status indicators on cards (e.g. low/normal/high/flood based on gage height thresholds)
- **ENH-03**: Timestamp showing when data was last fetched
- **ENH-04**: Offline mode — cache last-fetched data for use without internet

## Out of Scope

| Feature | Reason |
|---------|--------|
| Server or backend | Must remain a single static file |
| Build system / npm / bundler | Zero-dependency constraint |
| Historical data / charts | Current conditions only for v1 |
| Alerts or notifications | Passive display only |
| Multiple rivers or gauge networks | Chattahoochee-specific |
| User accounts or sharing | Personal use only |
| Mobile app | Browser-only |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| FILE-01 | Phase 1 | Pending |
| FILE-02 | Phase 1 | Pending |
| LAYOUT-01 | Phase 1 | Pending |
| LAYOUT-02 | Phase 1 | Pending |
| LAYOUT-03 | Phase 1 | Pending |
| DATA-01 | Phase 1 | Pending |
| DATA-02 | Phase 1 | Pending |
| DATA-03 | Phase 1 | Pending |
| DATA-04 | Phase 1 | Pending |
| DATA-05 | Phase 1 | Pending |
| DATA-06 | Phase 1 | Pending |
| CARD-01 | Phase 1 | Pending |
| CARD-02 | Phase 1 | Pending |
| CARD-03 | Phase 1 | Pending |
| DAM-01 | Phase 1 | Pending |
| DAM-02 | Phase 1 | Pending |

**Coverage:**
- v1 requirements: 16 total
- Mapped to phases: 16
- Unmapped: 0 ✓

---
*Requirements defined: 2026-05-17*
*Last updated: 2026-05-17 after initial definition*
