# Phase 2: Live Dashboard — Research

**Researched:** 2026-05-17
**Domain:** Browser-native JavaScript, USGS Instantaneous Values API, iframe embedding
**Confidence:** HIGH — all critical claims verified by live API calls and header inspection

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**File constraint (locked)**
- Single HTML file — all JavaScript must be inline in a `<script>` tag
- No external dependencies, no CDN, no imports
- Must work via `file://` protocol (no server)

**USGS API (locked)**
- Endpoint: USGS Instantaneous Values API (IV API) — public, CORS-friendly
- Fetch all 9 stations in a single API call using a comma-separated site list
- Parameters to fetch: 00065 (gage height, ft), 00060 (discharge, cfs), 00010 (temperature, °C), 63680 (turbidity, FNU)
- Temperature arrives in °C — convert to °F in JavaScript: `(C * 9/5) + 32`
- If a station doesn't report a parameter, omit that row from the card entirely (DATA-06)
- Station order is fixed (LAYOUT-02): 023344030, 02335000, 02335450, 02335815, 02335880, 02336000, 02337170, 02338000, 02338500

**Card behavior (locked)**
- CARD-01: Station name resolved from USGS API response (siteName field), with station ID shown
- CARD-02: Gage height (ft), discharge (cfs), temperature (°F), turbidity (FNU) where available
- CARD-03: Clicking any station card opens `https://waterdata.usgs.gov/monitoring-location/{stationId}/` in a new tab

**Dam section (locked)**
- DAM-01: Embed USACE Hydropower page via iframe: `https://spatialdata.usace.army.mil/Hydropower/`
- DAM-02: Graceful fallback — if iframe is blocked by X-Frame-Options, show a direct link instead

**Security (locked from Phase 1 threat model)**
- Use `textContent` (never `innerHTML`) when inserting USGS API response data into the DOM
- Station IDs and URLs are hardcoded author-controlled strings — not user input

### Claude's Discretion
- Fetch strategy: single multi-station IV API call vs. 9 parallel calls (single call preferred)
- Error handling granularity: per-station errors vs. page-level error
- Loading state: show placeholder dashes during fetch, replace on completion
- iframe fallback detection mechanism
- DOM manipulation approach

### Deferred Ideas (OUT OF SCOPE)
- ENH-01: Manual refresh button
- ENH-02: Color-coded status indicators
- ENH-03: Last-fetched timestamp
- ENH-04: Offline/cache mode
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| DATA-01 | Fetch live data from USGS IV API on page load for all 9 stations | Section 1: confirmed API URL and CORS headers |
| DATA-02 | Fetch gage height (00065, ft) | Section 1: parameter confirmed in live response |
| DATA-03 | Fetch discharge (00060, cfs) | Section 1: parameter confirmed in live response |
| DATA-04 | Fetch water temperature (00010, °C) and display as °F | Section 1: confirmed; stale-data gotcha documented |
| DATA-05 | Fetch turbidity (63680) for stations that report it | Section 1: 3 stations confirmed; others return no timeSeries entry |
| DATA-06 | Omit row entirely if station doesn't report the parameter | Section 1: absence = no timeSeries entry; see handling pattern |
| CARD-01 | Display station name from API and station ID | Section 3: siteName field confirmed; station-id span gap documented |
| CARD-02 | Show gage height, discharge, temperature, turbidity where available | Section 3: DOM pattern documented |
| CARD-03 | Clicking card opens USGS monitoring-location URL in new tab | Section 3: cursor+onclick pattern documented |
| DAM-01 | Embed USACE iframe | Section 2: URL confirmed; SAMEORIGIN block confirmed |
| DAM-02 | Fallback link if iframe blocked | Section 2: detection approach documented |
</phase_requirements>

---

## Summary

Phase 2 wires live USGS river data into the static DOM built by Phase 1. The JavaScript is straightforward: a single `fetch()` call to the USGS Instantaneous Values IV API returns a JSON payload containing all 9 stations and all 4 parameters in one response. The response shape is a flat `timeSeries[]` array where each entry covers one station + one parameter combination. Absent parameters simply have no entry in the array — the code must build a lookup map from the array and consult it per-station-per-parameter.

Two significant data gaps discovered by live API probing must be addressed in implementation. First, the station ID `023344030` listed in the project requirements and Phase 1 HTML is a typo — the real USGS site for Buford Dam is `02334430` (8 digits, not 9). The `023344030` ID returns zero results from the API. Second, station `02335815` (Morgan Falls) reports temperature, but its most recent temperature reading is from 2022-06-06 — the sensor appears inactive. The API returns this stale value with a `dateTime` timestamp four years in the past. The implementation must decide how to handle stale readings (see Pitfall 3).

The USACE iframe situation is confirmed: `spatialdata.usace.army.mil` sends `X-Frame-Options: SAMEORIGIN`, which blocks embedding from a `file://` origin. The browser fires the iframe `load` event regardless of whether the frame was actually blocked, so the `load` event cannot be used to detect success. The reliable detection technique uses a short `setTimeout` after the `load` event fires: attempt `iframe.contentDocument.location.href` in a `try/catch`; a `SecurityError` or an `about:blank` href confirms the block.

**Primary recommendation:** Build a `parsedData` lookup map keyed by `"USGS:{siteCode}:{paramCode}:00000"` from the `timeSeries[]` array. For each station card, query the map for each parameter; if the entry is absent or its value equals the noDataValue sentinel (`-999999`), hide the stat row. For the USACE iframe, default the fallback link to visible and hide it only if the `contentDocument` check succeeds.

---

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| USGS API fetch | Browser (JS) | — | Single `fetch()` call from inline `<script>`; no server involved |
| Response parsing / lookup | Browser (JS) | — | Pure in-memory transformation of JSON payload |
| DOM population | Browser (JS) | — | Targeted `querySelector` updates on the existing Phase 1 DOM |
| Temp °C → °F conversion | Browser (JS) | — | One-line arithmetic; no library needed |
| USACE iframe embed | Browser (HTML+JS) | — | Static `<iframe>` tag; JS adds fallback detection |
| Fallback link display | Browser (JS) | — | Show/hide based on iframe contentDocument check |
| Card click navigation | Browser (JS) | — | `onclick` or `cursor: pointer` + event delegation |

---

## 1. USGS Instantaneous Values API

### Confirmed API URL

[VERIFIED: live curl call 2026-05-17]

```
https://waterservices.usgs.gov/nwis/iv/?sites=02334430,02335000,02335450,02335815,02335880,02336000,02337170,02338000,02338500&parameterCd=00065,00060,00010,63680&format=json
```

**Critical correction:** The first station ID in REQUIREMENTS.md and in Phase 1 HTML is `023344030` — this ID returns zero results from the USGS IV API. The correct station ID for Buford Dam (Chattahoochee River at Buford Dam, Near Buford, GA) is `02334430` (8 digits). The implementation must use `02334430` in the API call and must also update the `data-station` attribute in the HTML from `023344030` to `02334430`.

**CORS headers confirmed** [VERIFIED: live curl call 2026-05-17]:
```
Access-Control-Allow-Origin: *
```
The API is fully CORS-open. `fetch()` from `file://` origin works without any issues.

### Response Structure

The top-level shape:
```
{
  "value": {
    "timeSeries": [ ...entries... ]
  }
}
```

Each `timeSeries` entry covers **one station + one parameter**:
```json
{
  "name": "USGS:02335000:00065:00000",
  "sourceInfo": {
    "siteName": "CHATTAHOOCHEE RIVER NEAR NORCROSS, GA",
    "siteCode": [{ "value": "02335000" }]
  },
  "variable": {
    "variableCode": [{ "value": "00065" }],
    "noDataValue": -999999.0
  },
  "values": [{
    "value": [{
      "value": "1.78",
      "qualifiers": ["P"],
      "dateTime": "2026-05-17T18:15:00.000-04:00"
    }]
  }]
}
```

**Key navigation path to the most recent reading:**
```javascript
const entry = timeSeries[i];
const reading = entry.values[0].value[0];  // most recent value
const numericValue = parseFloat(reading.value);
const timestamp = reading.dateTime;
```

### The `.name` Field as Parsing Key

The `.name` field follows the pattern `USGS:{siteCode}:{paramCode}:00000`. This is the reliable key for building a lookup map:

```javascript
// Build lookup map after fetch
const map = {};
data.value.timeSeries.forEach(ts => {
  map[ts.name] = ts;
});

// Access station 02335000 gage height:
const key = 'USGS:02335000:00065:00000';
const entry = map[key];  // undefined if station doesn't report this param
```

### Missing Parameters: Absence, Not Empty Arrays

[VERIFIED: live API probe 2026-05-17]

When a station does not report a parameter, **there is no timeSeries entry for that combination**. The `values[0].value` array is never empty in practice — the API omits the entry entirely. Detection:

```javascript
const key = `USGS:${stationId}:${paramCode}:00000`;
if (!map[key]) {
  // Station does not report this parameter — hide/remove the row
}
```

### Parameters Present Per Station (Live Data, 2026-05-17)

[VERIFIED: live curl call 2026-05-17]

| Station | 00065 (gage) | 00060 (discharge) | 00010 (temp) | 63680 (turbidity) |
|---------|-------------|------------------|-------------|------------------|
| 02334430 (Buford Dam) | yes | yes | yes | no |
| 02335000 (Norcross) | yes | yes | yes | yes |
| 02335450 (Roswell) | yes | yes | yes | no |
| 02335815 (Morgan Falls) | yes | yes | yes* | no |
| 02335880 (Powers Island) | yes | yes | yes | yes |
| 02336000 (Atlanta) | yes | yes | yes | yes |
| 02337170 (Fairburn) | yes | yes | yes | no |
| 02338000 (Whitesburg) | yes | yes | no | no |
| 02338500 (Franklin) | yes | yes | no | no |

*02335815 temperature: entry exists but dateTime is `2022-06-06T08:30:00.000-04:00` — sensor appears inactive. The value `17.4` is returned but is 4 years stale.

### noDataValue Sentinel

[VERIFIED: live API response 2026-05-17]

`noDataValue` is `-999999` (numeric). The API returns values as **strings**, so the check is:

```javascript
if (reading.value === '-999999') {
  // Sensor read but returned no-data sentinel
}
```

Treat a sentinel value identically to an absent timeSeries entry: hide the row.

---

## 2. USACE Iframe and Fallback

### X-Frame-Options Confirmed

[VERIFIED: live curl -I call 2026-05-17]

```
X-Frame-Options: SAMEORIGIN
```

The USACE Hydropower page at `https://spatialdata.usace.army.mil/Hydropower/` returns `X-Frame-Options: SAMEORIGIN`. When this page is loaded inside an iframe from a `file://` origin, the browser will block it — `file://` is not the same origin as `https://spatialdata.usace.army.mil`.

### Why `load` Event Cannot Detect the Block

[VERIFIED: jQuery issue #2672 investigation; confirmed by MDN X-Frame-Options docs]

The browser fires the iframe `load` event **even when X-Frame-Options blocks the content**. The error message appears in the DevTools console, but no `onerror` or blocked-`load` is dispatched to JavaScript. This means:

- `iframe.onload = fn` — fires regardless of success or block
- `iframe.onerror = fn` — does NOT fire for X-Frame-Options blocks (only for network errors)
- Inspecting `iframe.contentDocument` after load — throws `SecurityError` when blocked, returns the real document when successful

### Recommended Fallback Detection Pattern

The reliable technique for `file://` context:

```javascript
// In the DOM, default the fallback link to visible:
// <p id="dam-fallback"><a href="..." target="_blank">Open USACE Hydropower Schedule</a></p>
// <iframe id="dam-iframe" src="https://spatialdata.usace.army.mil/Hydropower/" ...></iframe>

const iframe = document.getElementById('dam-iframe');
const fallback = document.getElementById('dam-fallback');

iframe.addEventListener('load', function() {
  // Give browser time to complete XFO processing before checking
  setTimeout(function() {
    try {
      // If XFO blocked it, this throws SecurityError in all modern browsers
      // If it loaded successfully, this returns the real URL
      const href = iframe.contentDocument.location.href;
      if (href && href !== 'about:blank') {
        // Iframe loaded successfully — hide the fallback
        fallback.style.display = 'none';
      }
      // else: href is 'about:blank' = blocked, keep fallback visible
    } catch (e) {
      // SecurityError = blocked by XFO or cross-origin policy
      // Fallback link stays visible (it already is by default)
    }
  }, 100);
});
```

**Why default to fallback visible:** Since we know USACE sends SAMEORIGIN and the `file://` origin will always fail this check, the iframe will always be blocked in practice. Defaulting the fallback to visible means users see the link immediately without a 100ms flicker. The detection code would never hide it. For implementation simplicity, it is acceptable to skip the detection entirely and always show the fallback link alongside a styled note — this satisfies DAM-02 without the complexity of detection.

**Simpler alternative that always satisfies DAM-02:**
```html
<!-- Replace the placeholder <p> with: -->
<p id="dam-fallback">
  View the dam release schedule directly:
  <a href="https://spatialdata.usace.army.mil/Hydropower/" target="_blank" rel="noopener noreferrer">
    USACE Buford Dam Hydropower Schedule
  </a>
</p>
```
This always works, always satisfies DAM-02, and avoids the detection complexity. The iframe attempt can be omitted entirely or shown additionally below the link.

---

## 3. DOM Population Strategy

### The Real Phase 1 DOM (Confirmed by Reading hooch-snapshot.html)

[VERIFIED: read hooch-snapshot.html directly]

The Phase 1 HTML does NOT match the SKELETON.md description in two ways:

1. **Station names are already hardcoded as friendly names** in `.station-name` spans:
   `Buford Dam`, `Norcross`, `Roswell`, `Below Morgan Falls`, `Powers Island`, `Atlanta`, `Fairburn`, `Whitesburg`, `Franklin`
   The CONTEXT.md says CARD-01 requires resolving the station name from the API. The `siteName` field from the API returns uppercase names like `CHATTAHOOCHEE RIVER NEAR NORCROSS, GA`. The implementor must decide: use the API name (replacing the friendly Phase 1 name), or keep the Phase 1 friendly names and only update the station ID below them.

2. **No `.station-id` span exists** in the Phase 1 card headers. The SKELETON.md described one but it was not built. Phase 2 must either add it via JavaScript or the Phase 2 plan must modify the HTML directly.

3. **Stat value spans use class `.value`** (not `.stat-value` as the SKELETON described). The Phase 1 HTML has:
   ```html
   <div class="stat-row">
     <span class="label">Gage Height</span>
     <span class="value">— ft</span>
   </div>
   ```

4. **No `data-param` attributes** on `.stat-row` or `.value` elements. Phase 2 needs a strategy to identify which value span corresponds to which parameter.

### Recommended DOM Targeting Pattern

The cleanest approach: **add `data-param` attributes to each `.stat-row` in the HTML**, then query them in JavaScript. This requires modifying the Phase 1 HTML structure (which Phase 2 can do, since it modifies `hooch-snapshot.html`):

```html
<!-- Updated stat row HTML for Phase 2 -->
<div class="stat-row" data-param="00065">
  <span class="label">Gage Height</span>
  <span class="value">— ft</span>
</div>
<div class="stat-row" data-param="00060">
  <span class="label">Discharge</span>
  <span class="value">— cfs</span>
</div>
<div class="stat-row" data-param="00010">
  <span class="label">Temperature</span>
  <span class="value">— °F</span>
</div>
<!-- turbidity stations only: -->
<div class="stat-row" data-param="63680">
  <span class="label">Turbidity</span>
  <span class="value">— FNU</span>
</div>
```

JavaScript targeting:
```javascript
function populateCard(stationId, siteData) {
  const card = document.querySelector(`.card[data-station="${stationId}"]`);
  if (!card) return;

  // Update station name (use API siteName or keep friendly name — see open question below)
  // card.querySelector('.station-name').textContent = siteData.siteName;

  // Update each stat row
  card.querySelectorAll('.stat-row[data-param]').forEach(row => {
    const paramCode = row.dataset.param;
    const valueSpan = row.querySelector('.value');
    const key = `USGS:${stationId}:${paramCode}:00000`;
    const entry = siteData.map[key];

    if (!entry) {
      // Station doesn't report this parameter — remove row
      row.remove();
      return;
    }

    const reading = entry.values[0].value[0];
    if (!reading || reading.value === '-999999') {
      row.remove();
      return;
    }

    let displayValue = reading.value;
    let unit = '';

    if (paramCode === '00065') unit = ' ft';
    else if (paramCode === '00060') unit = ' cfs';
    else if (paramCode === '00010') {
      displayValue = ((parseFloat(reading.value) * 9 / 5) + 32).toFixed(1);
      unit = ' °F';
    }
    else if (paramCode === '63680') unit = ' FNU';

    valueSpan.textContent = displayValue + unit;
    valueSpan.style.color = 'var(--color-text)'; // change from muted to normal
  });
}
```

### Station Name Update (CARD-01 Open Question)

CARD-01 requires "station name resolved from USGS API." The Phase 1 HTML already has friendly short names that are more readable than the API's uppercase full names. Options:

**Option A:** Keep Phase 1 friendly names as-is; use them as the canonical station name (ignore the API siteName). Only add the station ID below in a new `<span class="station-id">`.

**Option B:** Replace with API `siteName` (all-caps, long). Requires title-casing or a station-name mapping.

**Option C:** Use API `siteName` but apply `text-transform: capitalize` or a title-case JS function.

Recommendation: **Option A is simplest** — the Phase 1 names are already correct. Add a `<span class="station-id">` below `.station-name` in each card header and populate it with the station ID from the `data-station` attribute. This satisfies CARD-01 (station name + ID shown).

### Card Click Behavior (CARD-03)

```javascript
// textContent constraint: station IDs are author-controlled strings — safe to use directly
document.querySelectorAll('.card[data-station]').forEach(card => {
  const stationId = card.dataset.station;
  card.style.cursor = 'pointer';
  card.addEventListener('click', function() {
    window.open(
      'https://waterdata.usgs.gov/monitoring-location/' + stationId + '/',
      '_blank',
      'noopener,noreferrer'
    );
  });
});
```

---

## 4. Error Handling

### API Failure (Network Error or HTTP Error)

When `fetch()` rejects (network down, DNS failure) or returns non-200:

**Recommended approach: per-card inline error state with a page-level banner**

Since all 9 stations are fetched in one call, an API failure affects all cards simultaneously. Show a single page-level error banner rather than updating all 9 cards individually:

```javascript
async function loadData() {
  try {
    const res = await fetch(API_URL);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const data = await res.json();
    processData(data);
  } catch (err) {
    showError('Unable to load river conditions. Check your internet connection.');
  }
}

function showError(message) {
  const banner = document.createElement('div');
  banner.setAttribute('role', 'alert');
  banner.style.cssText = 'background:var(--color-destructive);color:#fff;padding:var(--space-sm) var(--space-xl);font-size:var(--text-body);';
  banner.textContent = message;  // textContent — not innerHTML
  document.body.insertBefore(banner, document.body.firstChild);
}
```

**Cards retain placeholder dashes** (the Phase 1 state) on API failure — no DOM change needed for error state on individual cards.

### Stale Data (02335815 Morgan Falls Temperature)

The API returns the temperature entry for station `02335815` but with a `dateTime` from 2022-06-06. The value is `17.4` °C (63.3 °F). This is real data from the API — not a missing parameter — so the timeSeries entry exists.

Options:
1. **Display it with no special treatment** — show the stale reading as-is. Simple, but misleading.
2. **Define a staleness threshold** (e.g., > 24 hours old) and treat as missing, hiding the row.
3. **Display the reading but append a note** — but this requires DOM complexity and may confuse users.

**Recommendation: Option 2 with a 2-hour threshold.** Check `reading.dateTime` against `Date.now()`. If the reading is more than 2 hours old, treat the parameter as absent and remove the row.

```javascript
const STALE_THRESHOLD_MS = 2 * 60 * 60 * 1000; // 2 hours

function isStale(dateTimeStr) {
  return (Date.now() - new Date(dateTimeStr).getTime()) > STALE_THRESHOLD_MS;
}

// In the row population logic:
if (isStale(reading.dateTime)) {
  row.remove();
  return;
}
```

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| HTTP fetch | Custom XMLHttpRequest wrapper | `fetch()` native | Built into all modern browsers; no polyfill needed for modern-browser-only constraint |
| JSON parse | Manual string parsing | `response.json()` | Built into fetch API |
| CORS proxy | Reverse proxy server | Nothing needed | USGS returns `Access-Control-Allow-Origin: *` |
| Parameter lookup | Iterate timeSeries on every lookup | One-time lookup map | O(1) vs O(n) per card; build once after fetch |
| Temperature conversion | External library | `(C * 9/5) + 32` | One arithmetic expression |

---

## Common Pitfalls

### Pitfall 1: Invalid Station ID 023344030

**What goes wrong:** Using `023344030` in the API call returns zero results silently — the API returns HTTP 200 with an empty `timeSeries[]` array. No error, no exception. The Buford Dam card remains at placeholder dashes with no indication of why.

**Why it happens:** The project requirements contain a typo. The real USGS site number is `02334430` (8 digits). The requirements list `023344030` (9 digits, extra leading `0` inserted).

**How to avoid:** Use `02334430` in the fetch URL. Also update the `data-station` attribute in the HTML from `023344030` to `02334430` so DOM targeting works.

**Verification:** `https://waterservices.usgs.gov/nwis/iv/?sites=02334430&format=json` returns 3 timeSeries entries. `023344030` returns 0.

### Pitfall 2: `load` Event Does Not Indicate Iframe Success

**What goes wrong:** Adding `iframe.onload = () => fallback.style.display = 'none'` hides the fallback even when X-Frame-Options blocks the iframe.

**Why it happens:** Browsers fire `load` on the iframe element regardless of whether XFO allowed or blocked the content. The load event fires for both outcomes.

**How to avoid:** Use the `setTimeout + contentDocument` detection pattern described in Section 2. Or skip detection entirely and always show the fallback link (the simplest correct approach given SAMEORIGIN is always in effect for file:// context).

### Pitfall 3: Stale Data Returned as Valid Reading (02335815 Temperature)

**What goes wrong:** Station 02335815 (Morgan Falls) returns a temperature timeSeries entry, so `map[key]` is truthy and the row populates — but the reading is from 2022, so users see a temperature from 4 years ago displayed as current.

**Why it happens:** The USGS API returns the last reading it has, even if it's years old. The `values[0].value` array is not empty; the data just has a very old `dateTime`.

**How to avoid:** Check `reading.dateTime` against `Date.now()`. Apply a staleness threshold (2 hours recommended). If stale, treat as absent and remove the row.

### Pitfall 4: `innerHTML` with API Data

**What goes wrong:** Using `innerHTML` to set value spans causes XSS risk from the USGS API response.

**Why it happens:** API responses are JSON strings that could theoretically contain `<script>` or event handlers if the server were compromised or responses manipulated in transit.

**How to avoid:** Use `textContent` exclusively for all dynamic content insertion. This is a locked constraint from the Phase 1 threat model (T-01-07).

### Pitfall 5: Looking Up by Station ID but API Returns Corrected ID

**What goes wrong:** The `data-station` attribute in Phase 1 HTML says `023344030`. If the fetch URL uses `02334430` (corrected), the `.name` field in the API response will be `USGS:02334430:...`. A lookup keyed on `USGS:023344030:...` will miss all entries for this station.

**How to avoid:** Update both the HTML `data-station` attribute and the fetch URL to use `02334430`. The Phase 2 implementation modifies `hooch-snapshot.html`, so both changes happen together.

### Pitfall 6: noDataValue Appears as a Real Number

**What goes wrong:** `parseFloat('-999999')` is a valid float. A value row could display `-999999 ft` if the sentinel is not checked.

**Why it happens:** The noDataValue sentinel is a valid numeric string returned in the same `value` field as real readings.

**How to avoid:** Check `reading.value === '-999999'` (string comparison) before parsing. Remove the row if matched.

---

## Code Examples

### Full Fetch + Map Build Pattern

```javascript
// Source: USGS IV API verified 2026-05-17, structure from live response
const STATIONS = ['02334430','02335000','02335450','02335815','02335880','02336000','02337170','02338000','02338500'];
const PARAMS = ['00065','00060','00010','63680'];
const API_URL = 'https://waterservices.usgs.gov/nwis/iv/?sites=' +
  STATIONS.join(',') + '&parameterCd=' + PARAMS.join(',') + '&format=json';

const STALE_MS = 2 * 60 * 60 * 1000; // 2 hours

async function loadRiverData() {
  const res = await fetch(API_URL);
  if (!res.ok) throw new Error('HTTP ' + res.status);
  const json = await res.json();

  // Build O(1) lookup map
  const map = {};
  json.value.timeSeries.forEach(ts => { map[ts.name] = ts; });

  return map;
}
```

### Reading a Single Parameter Value

```javascript
// Source: live API response structure verified 2026-05-17
function getReading(map, stationId, paramCode) {
  const key = `USGS:${stationId}:${paramCode}:00000`;
  const ts = map[key];
  if (!ts) return null; // station doesn't report this parameter

  const reading = ts.values[0].value[0];
  if (!reading) return null;
  if (reading.value === '-999999') return null; // noDataValue sentinel
  if ((Date.now() - new Date(reading.dateTime).getTime()) > STALE_MS) return null; // stale

  return reading.value; // string, e.g. "1.78"
}
```

### Temperature Conversion

```javascript
// (C * 9/5) + 32, displayed to 1 decimal place
function celsiusToFahrenheit(cStr) {
  return ((parseFloat(cStr) * 9 / 5) + 32).toFixed(1);
}
```

### DOM Update for One Card

```javascript
// Source: Phase 1 DOM structure, verified by reading hooch-snapshot.html
function populateCard(stationId, map) {
  const card = document.querySelector(`.card[data-station="${stationId}"]`);
  if (!card) return;

  card.querySelectorAll('.stat-row[data-param]').forEach(row => {
    const paramCode = row.dataset.param;
    const rawValue = getReading(map, stationId, paramCode);

    if (rawValue === null) {
      row.remove(); // DATA-06: remove row entirely if no data
      return;
    }

    const valueSpan = row.querySelector('.value');
    let display = rawValue;
    let unit = '';
    if (paramCode === '00065') unit = ' ft';
    else if (paramCode === '00060') unit = ' cfs';
    else if (paramCode === '00010') { display = celsiusToFahrenheit(rawValue); unit = ' °F'; }
    else if (paramCode === '63680') unit = ' FNU';

    valueSpan.textContent = display + unit; // textContent — never innerHTML
    valueSpan.style.color = 'var(--color-text)';
  });
}
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| XMLHttpRequest for API calls | `fetch()` + async/await | ~2015-2017 (now baseline) | Simpler, Promise-based, no extra setup |
| iframe `onload` as success signal | `setTimeout` + `contentDocument` check | Documented behavior since ~2014 | `onload` fires even on block; detection requires content access attempt |

---

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | Station `02335815` temperature sensor is genuinely inactive (stale from 2022) rather than a temporary outage | Pitfall 3 / DATA-04 | If sensor recovers, the 2-hour threshold will correctly show data — no harm |
| A2 | The `02335815` stale temperature reading will remain stale at implementation time | Data table in Section 1 | Could return current data by then — code handles both correctly |
| A3 | The USACE `X-Frame-Options: SAMEORIGIN` header will remain in place | Section 2 | If USACE removes XFO, the iframe loads and `contentDocument` check shows fallback unnecessary |
| A4 | Station `02338000` and `02338500` will continue not reporting temperature | Data table in Section 1 | If they add temp sensors, code handles it correctly since the timeSeries entry would appear |

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| `fetch()` API | DATA-01 (USGS call) | Yes (modern browsers) | Baseline | None needed — modern browser assumed |
| USGS waterservices.usgs.gov | DATA-01 | Confirmed reachable | — | Show error banner |
| spatialdata.usace.army.mil | DAM-01 | Confirmed reachable | — | Fallback link (DAM-02) |
| Internet connection | All data | Required at page-open | — | Error banner; placeholder dashes stay |

---

## Validation Architecture

nyquist_validation is enabled per `.planning/config.json`.

### Test Framework

No automated test framework is appropriate for this phase. The deliverable is a single HTML file that runs entirely in a browser. Verification is done by:
1. Opening `hooch-snapshot.html` in a browser
2. Observing network requests in DevTools
3. Confirming card values populate

There is no Node.js test runner applicable to vanilla browser JavaScript loaded via `file://`.

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Command |
|--------|----------|-----------|---------|
| DATA-01 | API fetched on load | manual | Open file, check DevTools Network tab for waterservices.usgs.gov request |
| DATA-02 | Gage height shows in ft | manual | Inspect any card's gage height value |
| DATA-03 | Discharge shows in cfs | manual | Inspect any card's discharge value |
| DATA-04 | Temperature shows in °F | manual | Verify value is > 32 and < 120 (reasonable range) |
| DATA-05 | Turbidity on 3 stations only | manual | Count turbidity rows across all 9 cards |
| DATA-06 | No row for absent params | manual | Confirm 02338000 has no Temperature row |
| CARD-01 | Station name and ID | manual | Verify header band has name + numeric ID |
| CARD-02 | All 4 params where applicable | manual | Spot-check values against USGS website |
| CARD-03 | Click opens USGS URL in new tab | manual | Click a card, verify URL in new tab |
| DAM-01 | Iframe or fallback shown | manual | Confirm dam section has content (not placeholder "will appear here") |
| DAM-02 | Fallback link visible | manual | Confirm fallback link is present and clickable |

### Wave 0 Gaps

None — no test infrastructure is applicable to this phase. All verification is manual browser testing.

---

## Security Domain

`security_enforcement: true`, ASVS level 1 per `.planning/config.json`.

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | No | No auth in this app |
| V3 Session Management | No | No sessions |
| V4 Access Control | No | No access control |
| V5 Input Validation | Yes (data from USGS API) | `textContent` only (locked from Phase 1 threat model T-01-07) |
| V6 Cryptography | No | No credentials or secrets |

### Known Threat Patterns for This Stack

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| API response injection | Tampering / Spoofing | `textContent` exclusively; no `innerHTML`; already locked by Phase 1 threat model |
| Open redirect via station ID | Elevation of Privilege | Station IDs are hardcoded author constants in a JS array; not user input; no injection path |
| iframe clickjacking from embedded USACE page | Spoofing | Not applicable — this app embeds the iframe, it doesn't run inside one |

**Phase 2 threat model note:** The only new attack surface in Phase 2 is the USGS API response being inserted into the DOM. The `textContent` constraint eliminates this surface. Station IDs and the USGS monitoring-location URL are hardcoded constants, not constructed from any external input.

---

## Open Questions (RESOLVED)

1. **Station name strategy (CARD-01)**
   - What we know: Phase 1 hardcoded short friendly names (`Norcross`, `Roswell`, etc.). CONTEXT.md says CARD-01 requires "station name resolved from USGS API." The API returns uppercase full names (`CHATTAHOOCHEE RIVER NEAR NORCROSS, GA`).
   - What's unclear: Does CARD-01 require using the API name (replacing friendly names), or does showing the numeric station ID below the existing friendly name satisfy the requirement?
   - Recommendation: Keep the Phase 1 friendly names; satisfy CARD-01 by adding the numeric station ID below in a `.station-id` span. The requirement says "station name (resolved from USGS API)" but the Phase 1 names are accurate. If the planner interprets this strictly, a name-mapping step would be needed.

2. **Station ID correction scope**
   - What we know: Phase 1 HTML has `data-station="023344030"`. The correct USGS ID is `02334430`.
   - What's unclear: Does this require a requirements update, or is it handled silently as an implementation correction?
   - Recommendation: Treat as an implementation correction. Update the `data-station` attribute and the fetch URL silently in the Phase 2 task. Flag it in the plan for user awareness.

---

## Sources

### Primary (HIGH confidence)
- Live `curl` to `https://waterservices.usgs.gov/nwis/iv/` — response structure, CORS headers, parameter availability per station, noDataValue sentinel, `.name` field format
- Live `curl -I` to `https://spatialdata.usace.army.mil/Hydropower/` — X-Frame-Options: SAMEORIGIN header confirmed
- `hooch-snapshot.html` — actual Phase 1 DOM structure: `.value` class, no `.station-id` span, friendly names already present

### Secondary (MEDIUM confidence)
- [MDN Web Docs: X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) — XFO behavior and browser enforcement
- [jQuery issue #2672](https://github.com/jquery/jquery/issues/2672) — confirmed `load` event fires even when XFO blocks the iframe

### Tertiary (LOW confidence)
- [Siderite's Blog: Detecting if URL can be loaded in iframe](https://siderite.dev/blog/detecting-if-url-can-be-loaded-in-iframe.html) — `contentDocument.location.href` detection pattern; browser-specific behavior details

---

## Metadata

**Confidence breakdown:**
- USGS API URL, CORS, response structure: HIGH — verified by live API calls
- Per-station parameter availability: HIGH — verified by live API calls on 2026-05-17
- iframe X-Frame-Options behavior: HIGH — header confirmed by live curl; `load` event behavior confirmed by jQuery issue #2672 and MDN docs
- iframe `contentDocument` detection pattern: MEDIUM — standard pattern from multiple sources; browser-specific edge cases possible
- Stale data handling (02335815): HIGH — observed directly in API response

**Research date:** 2026-05-17
**Valid until:** 2026-06-17 (stable API; verify parameter availability if implementing more than 30 days out — sensor availability can change)
