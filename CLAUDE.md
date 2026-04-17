# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flag-It is a **vanilla HTML/CSS/JS Progressive Web App (PWA)** for tracking bouldering sessions and hangboard training on mobile. There is no build system, no package manager, and no test framework — the entire application lives in a single `index.html` file.

## Development Workflow

**To develop:** Open `index.html` directly in a browser. No build step required.

**To deploy:** Push to `main` — GitHub Actions (`.github/workflows/deploy.yml`) automatically deploys to GitHub Pages.

**Service worker cache:** Bump the cache name in `sw.js` (`flagit-v3` → `flagit-v4`, etc.) whenever making changes that need to invalidate cached assets for existing users.

## Architecture

### Single-file structure

`index.html` contains everything in order:
1. `<style>` block (lines ~15–264) — all CSS, dark theme, accent color `#7f77dd`
2. HTML markup (lines ~266–453) — six page `<div>` sections
3. `<script>` block (lines ~455–1099+) — all application logic

External dependencies loaded from CDN:
- **Chart.js v4.4.1** — for progress/grade/hangtime charts
- **Google Fonts** — Barlow font family

### Six pages (bottom nav)

| Page | Element ID | Purpose |
|---|---|---|
| Home/Feed | `#page-home` | Recent sessions, filtering |
| Log Session | `#page-log` | Record bouldering sessions |
| Stats | `#page-stats` | Analytics, PRs, charts |
| Goals | `#page-goals` | Grade goals and count goals |
| Hangboard | `#page-hang` | Hang training logging + timer |
| Gyms | `#page-gyms` | Manage gyms, backup/restore data |

Navigation is driven by `showPage(pageId)`.

### Data model

All state lives in four global arrays, persisted to localStorage:

| Variable | localStorage key | Contents |
|---|---|---|
| `sessions` | `flagit_sessions_v2` | Bouldering session records |
| `gyms` | `flagit_gyms_v1` | Gym list with favorite flag |
| `goals` | `flagit_goals_v2` | Grade goals and count goals |
| `hangSessions` | `flagit_hang_v1` | Hangboard training records |

**Mutation pattern:** Mutate the array directly → call the corresponding save function (`save()`, `saveGyms()`, `saveGoals()`, `saveHang()`). There is no reactive binding.

### Key constants (hardcoded in JS)

- `GRADES` — 20 climbing grades from `'4'` to `'8b+'`, each with a hex color
- `TYPES` — 9 route types (Slab, Vertical, Overhang, etc.)
- `GRIPS` — 9 grip types for hangboard (Crimp, Jug, Open hand, etc.)

### Rendering pattern

All UI updates use direct `element.innerHTML = templateLiteralString` assignments. There is no virtual DOM or templating library. Modals/overlays are created dynamically and dismissed on background click.

### Charts

Three Chart.js instances (`progressChart`, `gradesChart`, `hangChart`) are destroyed with `.destroy()` before being re-created to prevent memory leaks.

### Session editing

Uses two globals: `editMode` (boolean) and `editSessionId` (string). The save form function checks these to decide whether to insert or update a record.

## Conventions

- **XSS protection:** User-supplied strings must be HTML-escaped before insertion into `innerHTML`. Use the existing `escHtml()` utility function for this.
- **Mobile-first:** The layout targets max-width 430px. Use CSS `env(safe-area-inset-*)` for fixed bottom elements.
- **PWA offline:** All new static assets must be listed in the `sw.js` cache array and the cache version bumped.
- **No external state or frameworks** — keep logic self-contained in the script block; avoid introducing npm dependencies or build tools.
