# Phase 2: Live Dashboard — Context

**Gathered:** 2026-05-17
**Status:** Ready for planning
**Source:** REQUIREMENTS.md + ROADMAP.md (fully specified)

<domain>
## Phase Boundary

Phase 2 wires live data into the static shell produced by Phase 1. Opening `hooch-snapshot.html` triggers JavaScript that fetches live USGS river conditions for all 9 Chattahoochee stations and populates the cards. The Buford Dam section embeds the USACE Hydropower iframe with a visible fallback link if X-Frame-Options blocks it. Each station card becomes clickable, opening the USGS monitoring-location page in a new tab.

Phase 2 is the final v1 phase — nothing ships after this.

</domain>

<decisions>
## Implementation Decisions

### File constraint (locked)
- Single HTML file — all JavaScript must be inline in a `<script>` tag
- No external dependencies, no CDN, no imports
- Must work via `file://` protocol (no server)

### USGS API (locked)
- Endpoint: USGS Instantaneous Values API (IV API) — public, CORS-friendly
- Fetch all 9 stations in a single API call using a comma-separated site list
- Parameters to fetch: 00065 (gage height, ft), 00060 (discharge, cfs), 00010 (temperature, °C), 63680 (turbidity, FNU)
- Temperature arrives in °C — convert to °F in JavaScript: `(C * 9/5) + 32`
- If a station doesn't report a parameter, omit that row from the card entirely (DATA-06)
- Station order is fixed (LAYOUT-02): 023344030, 02335000, 02335450, 02335815, 02335880, 02336000, 02337170, 02338000, 02338500

### Card behavior (locked)
- CARD-01: Station name resolved from USGS API response (siteName field), with station ID shown
- CARD-02: Gage height (ft), discharge (cfs), temperature (°F), turbidity (FNU) where available
- CARD-03: Clicking any station card opens `https://waterdata.usgs.gov/monitoring-location/{stationId}/` in a new tab

### Dam section (locked)
- DAM-01: Embed USACE Hydropower page via iframe: `https://spatialdata.usace.army.mil/Hydropower/`
- DAM-02: Graceful fallback — if iframe is blocked by X-Frame-Options, show a direct link instead

### Security (locked from Phase 1 threat model)
- Use `textContent` (never `innerHTML`) when inserting USGS API response data into the DOM (XSS prevention, T-01-07 from Phase 1 threat model)
- Station IDs and URLs are hardcoded author-controlled strings — not user input

### Claude's Discretion
- Fetch strategy: single multi-station IV API call vs. 9 parallel calls (single call preferred — simpler, fewer requests)
- Error handling granularity: per-station errors vs. page-level error
- Loading state: show placeholder dashes during fetch, replace on completion
- iframe fallback detection mechanism (onerror vs. postMessage vs. try-catch)
- DOM manipulation approach (querySelectorAll by data-station attribute is the natural hook from Phase 1 HTML)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Phase 1 deliverable
- `.planning/phases/01-static-shell/01-01-PLAN.md` — DOM structure, data-station attributes, card anatomy, CSS custom properties
- `.planning/phases/01-static-shell/01-UI-SPEC.md` — visual contract; stat row structure Phase 2 must populate
- `hooch-snapshot.html` — the actual file being modified in Phase 2

### Requirements
- `.planning/REQUIREMENTS.md` — DATA-01 through DATA-06, CARD-01 through CARD-03, DAM-01, DAM-02

### Project
- `.planning/ROADMAP.md` — Phase 2 success criteria (5 items)
- `.planning/PROJECT.md` — core constraints and delivery model

</canonical_refs>

<specifics>
## Specific Ideas

- USGS IV API multi-site URL pattern: `https://waterservices.usgs.gov/nwis/iv/?sites=023344030,02335000,...&parameterCd=00065,00060,00010,63680&format=json`
- Station IDs (exact, in order): 023344030, 02335000, 02335450, 02335815, 02335880, 02336000, 02337170, 02338000, 02338500
- Turbidity stations only: 02335000, 02335880, 02336000 — cards for all others already lack the Turbidity row from Phase 1
- USACE URL: `https://spatialdata.usace.army.mil/Hydropower/`
- USGS monitoring-location URL pattern: `https://waterdata.usgs.gov/monitoring-location/{stationId}/`

</specifics>

<deferred>
## Deferred Ideas

- ENH-01: Manual refresh button
- ENH-02: Color-coded status indicators
- ENH-03: Last-fetched timestamp
- ENH-04: Offline/cache mode

All deferred to v2 — out of scope for this phase.

</deferred>

---

*Phase: 02-live-dashboard*
*Context gathered: 2026-05-17 from REQUIREMENTS.md + ROADMAP.md*
