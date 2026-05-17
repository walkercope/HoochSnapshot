<!-- GSD:project-start source:PROJECT.md -->

## Project

**HoochSnapshot**

A single self-contained HTML file that displays live Chattahoochee River conditions — gage height, discharge, water temperature (°F), and turbidity — pulled directly from USGS Water Services on page load. It also embeds the USACE Hydropower schedule for Buford Dam. Opens in any browser, no server required.

**Core Value:** Open the file and immediately know current river conditions at each gauge — no setup, no login, no dependencies.

### Constraints

- **Tech stack**: Plain HTML/CSS/JavaScript — no frameworks, no bundlers, no npm. Must remain a single file.
- **Data access**: USGS API is public and CORS-friendly. USACE iframe may be blocked by X-Frame-Options — handle gracefully.
- **Deployment**: None. File lives locally and is opened directly in a browser.

<!-- GSD:project-end -->

<!-- GSD:stack-start source:STACK.md -->

## Technology Stack

Technology stack not yet documented. Will populate after codebase mapping or first phase.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->

## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->

## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->

## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->

## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:

- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->

## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
