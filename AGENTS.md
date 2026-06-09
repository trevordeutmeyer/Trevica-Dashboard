# Codex / AI Agent Working Notes

This is a personal family dashboard for a non-developer owner. Prioritize reliability, clear documentation, and changes that can be verified without a build pipeline.

## Product Intent

- Primary device: Surface Pro in portrait orientation, running in kiosk mode.
- Primary use: walk-by household status at a glance from 3–6 feet.
- Secondary use: tap into chores, approvals, history, and management from any family device.
- Audience: family members including young kids. Tone is casual, warm, and energetic.
- Responsive: layout adapts for phone portrait, phone landscape, tablet portrait, and desktop landscape.

## Engineering Rules

- Keep the app deployable on GitHub Pages with no build step.
- Do not add a backend unless the owner explicitly approves it.
- Prefer Firebase client SDK plus secure Realtime Database rules.
- Do not commit Firebase Admin SDK service account files or private keys.
- Firebase client config is allowed in the frontend. Security must come from rules and auth.
- Keep changes small and verifiable. One logical change per session when possible.
- After frontend edits, verify with `python -m http.server 8080` and check browser console.

## Current Shape

- `index.html` contains all HTML, CSS, and JavaScript — no separate files.
- Firebase state path: `familyDashboard/state`.
- Local fallback key: `fb_v11`.
- Google Calendar iframe URL configured in the `GOOGLE_CAL_URL` constant.
- Parent PIN: `PARENT_PIN = '1234'` (hard-coded, visual deterrent only).

## State Schema

State is normalized through `freshState()` → `normalizeState()` → `normalizeSettings()` / `normalizePerson()` on every load. Migrations run inside `normalizeState` and are gated by one-time flags (e.g. `streakResetDone`). New fields must be added to both `freshState()` and the `normalizeState` return object.

Key top-level fields:

```
schemaVersion, household, people[], checked{}, approved{}, pending[],
activity[], logs[], globalTodos[], weekKey, settings{}, dinnerEnabled,
dinnerMenu, customBanners[], streakResetDone, meta{}
```

## Rendering Pipeline

`renderAll()` calls:
- `renderUnifiedFrontMatrix()` — kids hero cards + adults compact strip
- `renderManageView()` — family settings panel
- `updateSystemCounters()` — badge counts, stats strip
- `processSystemBanners()` — banner priority logic

Always call `saveState(reason)` then `renderAll()` after mutating state.

## Banner Priority Order

1. Hard-coded movie-time alert (7:35–8:05 PM)
2. Active custom banners (from `state.customBanners[]`, most recently started wins)
3. Dinner menu (if `state.dinnerEnabled && state.dinnerMenu`)
4. Rotating welcome messages (7 messages, index by date+hour)

## Kiosk Auto-Update

An IIFE before the boot line polls the GitHub Pages URL via HEAD request every 10 minutes, comparing `ETag`/`Last-Modified` headers. On change detection:
- If idle > 45 s: auto-reload after 1.8 s.
- Otherwise: show tap-to-update bar at bottom of screen.
- Nightly 3 AM hard reload is also scheduled.

## Responsive Breakpoints

```
< 600 px width              → phone portrait: single-column, scrollable
600–767 px width            → large phone portrait: 2-col kids
768–1099 px width           → tablet portrait / kiosk: default layout
≥ 1100 px width             → desktop: two-column grid layout
height < 520 px + landscape → phone landscape: compact two-column, scrollable
```

## Near-Term Refactor Direction

Do not split files just for style. Split only when making a substantive change that makes the single-file unworkable:

- `src/state.js` — Firebase/local state logic
- `src/render.js` — UI rendering
- `src/actions.js` — state mutations
- `src/config.js` — safe public config and constants
- `styles.css` — styling

If adding a build step, prefer Vite and keep GitHub Pages deployment simple.
