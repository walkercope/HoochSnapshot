# Walking Skeleton: HoochSnapshot

**Phase:** 1 — Static Shell
**Created:** 2026-05-17

---

## What This Is

The Walking Skeleton is the thinnest possible slice that can be opened in a browser and shows the full intended structure. For HoochSnapshot, the skeleton IS the final Phase 1 deliverable — a single `.html` file with no external dependencies that renders the complete layout shell.

There is no server. There is no build system. Opening the file in a browser is the entire deployment model.

---

## Architectural Decisions (Locked for All Future Phases)

These decisions are made in Phase 1 and must not be renegotiated in Phase 2.

| Decision | Value | Rationale |
|----------|-------|-----------|
| Delivery unit | Single `.html` file | Core project constraint (FILE-01, FILE-02) |
| CSS location | `<style>` block inside the `.html` file | No external stylesheets — zero-dependency constraint |
| JS location | `<script>` block inside the `.html` file | No external scripts — zero-dependency constraint |
| Font | `system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif` | No web fonts, no CDN |
| CSS tokens | CSS custom properties declared at `:root` | Consistent theming without a preprocessor |
| Color scheme | Dark (`--color-bg: #0f1117`) | Dashboard readability |
| Layout engine | CSS Grid (`auto-fill, minmax(280px, 1fr)`) | Responsive card grid without a framework |
| Station order | Fixed in DOM: 023344030 → 02338500 (9 stations, exact order) | LAYOUT-02; order never changes |
| Turbidity rows | Present only on 02335000, 02335880, 02336000 cards | DATA-05; structure set in Phase 1, wired in Phase 2 |

---

## Stack

| Layer | Technology |
|-------|-----------|
| Markup | HTML5 |
| Styles | CSS3 (inline `<style>` block) |
| Scripting | Vanilla JavaScript (inline `<script>` block — Phase 2 only) |
| Build | None |
| Package manager | None |
| External dependencies | None |
| Deployment | File system (open in browser) |

---

## Directory Layout

```
HoochSnapshot/
  hooch-snapshot.html     — the entire application (created in Phase 1)
  .planning/              — GSD planning artifacts (not shipped)
```

No subdirectories. No `src/`. No `dist/`. The project root IS the deliverable root.

---

## File Naming

The output file is `hooch-snapshot.html`. Lowercase, hyphenated. Opening this file in any modern browser is the complete user experience.

---

## CSS Custom Properties Contract

All properties declared at `:root`. Phase 2 must use these — do not introduce new color or spacing values without adding a custom property here.

### Spacing
```
--space-xs:  4px
--space-sm:  8px
--space-md:  16px
--space-lg:  24px
--space-xl:  32px
--space-2xl: 48px
--space-3xl: 64px
```

### Color
```
--color-bg:           #0f1117
--color-surface:      #1c1f26
--color-surface-alt:  #252830
--color-accent:       #3b82f6
--color-border:       #2e3138
--color-text:         #e2e8f0
--color-text-muted:   #8b92a5
--color-destructive:  #ef4444
```

### Typography
```
--text-body:    14px
--text-label:   12px
--text-heading: 18px
--text-display: 24px
```

---

## DOM Skeleton

```
html
  head
    meta charset, viewport
    title
    style (all CSS inline)
  body
    header
      "HoochSnapshot"  (--text-label, --color-text-muted)
      "Chattahoochee River Conditions"  (--text-label, --color-text-muted)
    main
      section#dam
        h2 "Buford Dam Release Schedule"
        p "Dam release schedule will appear here."
      section#stations
        div.card[data-station="023344030"]  — 3 stat rows
        div.card[data-station="02335000"]   — 4 stat rows (turbidity)
        div.card[data-station="02335450"]   — 3 stat rows
        div.card[data-station="02335815"]   — 3 stat rows
        div.card[data-station="02335880"]   — 4 stat rows (turbidity)
        div.card[data-station="02336000"]   — 4 stat rows (turbidity)
        div.card[data-station="02337170"]   — 3 stat rows
        div.card[data-station="02338000"]   — 3 stat rows
        div.card[data-station="02338500"]   — 3 stat rows
```

---

## Phase 2 Extension Points

Phase 2 adds behavior by extending this skeleton. Architectural anchors:

| Anchor | How Phase 2 Uses It |
|--------|---------------------|
| `data-station` attribute on each `.card` | JavaScript selects cards by station ID to inject live data |
| `.stat-value` spans (set up in Phase 1) | Phase 2 replaces `—` text with fetched values |
| `section#dam` | Phase 2 inserts `<iframe>` or fallback `<a>` inside this section |
| Inline `<script>` block | Phase 2 adds USGS fetch logic here |
| Turbidity rows already in DOM for 3 stations | Phase 2 populates them; cards for other stations never get turbidity rows added |

---

*Walking Skeleton created: 2026-05-17*
