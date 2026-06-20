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
- **Never use `open(file,'w').write(...)` to overwrite `index.html`.** Always use an atomic Python read → string replace → write pattern. The Edit tool can silently truncate large files mid-session.

## Current Shape

- `index.html` contains all HTML, CSS, and JavaScript — no separate files.
- Firebase state path: `familyDashboard/state`.
- Local fallback key: `fb_v11`.
- Google Calendar iframe URL configured in the `GOOGLE_CAL_URL` constant.
- Parent PIN: `PARENT_PIN = '1234'` (hard-coded, visual deterrent only).

## State Schema

State is normalized through `freshState()` → `normalizeState()` → `normalizeSettings()` / `normalizePerson()` / `normalizeChecked()` on every load. Migrations run inside `normalizeState` and are gated by one-time flags (e.g. `streakResetDone`). New fields must be added to both `freshState()` and the `normalizeState` return object. New settings fields must also be added to `normalizeSettings`.

Key top-level fields:

```
schemaVersion, household, people[], checked{}, approved{}, pending[],
activity[], logs[], globalTodos[], weekKey, settings{}, dinnerEnabled,
dinnerMenu, customBanners[], streakResetDone, meta{}
```

### settings fields

```
dinnerEnabled (bool), dinnerMenu (string), motionWake (bool)
```

> **Note on motionWake persistence**: Edge kiosk mode runs InPrivate — localStorage is wiped on every session reset. `toggleMotionWake(on)` therefore **dual-writes**: to `state.settings.motionWake` + Firebase (survives InPrivate wipe, server-side) AND to `localStorage` key `'motionWake'` (faster read for normal browser sessions). `getMotionWakeSetting()` checks `state.settings.motionWake` first (Firebase-authoritative), then localStorage as fallback. `applyRemoteState` auto-calls `startMotionWake()` when Firebase delivers `motionWake=true` so the camera restarts automatically on each new kiosk session without user interaction.

### weekKey

`weekKey` is a `YYYY-MM-DD` string — the Monday of the current ISO week (e.g. `"2026-06-15"`). It is **always recalculated** from `getWeekKey()` on load; the stored value is never trusted.

> Previous format `w2026_25` is obsolete. It had a time-of-day bug where Mon–Fri and Sat–Sun of the same week returned different week numbers.

### checked schema

`state.checked[personId]` is a date-stamped dict, not an array:

```js
{ "j1": "2026-06-15", "j2": "2026-06-17" }
```

Value = `YYYY-MM-DD` date the chore was last completed. `normalizeChecked` migrates old array format automatically on first load.

### Chore schema

```js
{ id, name, type: 'weekly'|'job', amt: 0, days: [0-6, ...] }
```

`days` is optional. When present and non-empty, the chore is "day-specific" (resets daily). When absent or empty, it is "unscheduled weekly" (resets each Monday).

## Key Functions

### Week key helpers

```js
function _mondayOf(d)           // 'YYYY-MM-DD' of the Monday of d's week
function getWeekKey()           // Monday of current week
function getWeekMondayStr()     // alias of getWeekKey()
function dateToWeekKey(dateStr) // Monday of the week containing dateStr
```

ISO week: Mon = start, Sun = end. All days Mon–Sun map to the same weekKey.

### isChecked(pid, cid)

```
if chore.days is non-empty:
    return stored date === todayStr()          // day-specific: resets daily
else:
    return dateToWeekKey(stored) === weekKey   // unscheduled: resets weekly
```

### wasCheckedThisWeek(pid, cid)

```
return dateToWeekKey(stored) === weekKey
```

Used for allowance/progress tracking — counts any completion this week regardless of whether the chore has already reset for today.

### toggleChore(pid, cid)

On check: stamps `state.checked[pid][cid] = todayStr()`, plays chime + confetti, pushes activity log entry. On uncheck: deletes the stamp, revokes allowance approval if present.

## Rendering Pipeline

`renderAll()` calls:
- `renderUnifiedFrontMatrix()` — 2×2 kids/adults hero grid
- `renderManageView()` — family settings panel
- `updateSystemCounters()` — badge counts, stats strip
- `processSystemBanners()` — banner priority logic

Always call `saveState(reason)` then `renderAll()` after mutating state.

## Layout — 2×2 Hero Grid

All four family members render as equal-size cards in a CSS grid:

```
[ Henry  | Jules ]
[ Mom    | Dad   ]
```

Kids cards show: emoji, name, streak badge, allowance progress bar, chore rows.
Adult cards show: emoji, name, chore/task rows (no allowance or streak).

## Banner Priority Order

1. Hard-coded movie-time alert (7:35–8:05 PM)
2. Active custom banners (from `state.customBanners[]`, most recently started wins)
3. Dinner menu (if `state.dinnerEnabled && state.dinnerMenu`)
4. Rotating welcome messages (7 messages, index by date+hour)

## Motion Wake

Controlled by `state.settings.motionWake` (Firebase) with `localStorage` as fallback. Read via `getMotionWakeSetting()`. Written via `toggleMotionWake(on)` which dual-writes to both Firebase and localStorage. Auto-started from `applyRemoteState` on each Firebase sync (handles kiosk InPrivate session resets). When `true`:
- `getUserMedia` opens a low-res camera stream.
- Each frame is diffed against the previous (grayscale, `DIFF_THRESH=18`, `CHANGED_FRAC=0.003`).
- No motion for `IDLE_MS=120000` (2 min) → sleep overlay shown.
- Motion detected → sleep overlay dismissed.
- `navigator.wakeLock.request('screen')` prevents OS sleep.

Ghost-click guard: when the PIN modal closes and `switchTab('manage')` is called, there is a 400 ms delay before the manage panel opens to avoid the phantom touch event hitting the back button.

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

## Known Gotchas

- **Edit tool truncation**: the Edit tool can leave stale cached content on large files. Use Python atomic read → replace → write for all multi-block changes to `index.html`.
- **Ghost click**: touch screens fire a ~300 ms delayed click after a touch. When closing a modal that reveals another interactive element, delay the reveal by at least 400 ms.
- **`#panel-manage.active`**: uses `position: fixed; inset: 0` to guarantee full-screen regardless of flex layout ancestry.
- **normalizeSettings must include all settings fields**: any field omitted from the return value is silently dropped on every state reload.
- **weekKey migration**: old `w20XX_NN` format keys stored in Firebase are harmless — `normalizeState` overwrites `weekKey` with the new format on every load.
- **Git index.lock**: the sandbox cannot remove `.git/index.lock`. If a commit fails with this error, run `del .git\index.lock` from the user's Windows terminal, then `git commit && git push`.

## Near-Term Refactor Direction

Do not split files just for style. Split only when making a substantive change that makes the single-file unworkable:

- `src/state.js` — Firebase/local state logic
- `src/render.js` — UI rendering
- `src/actions.js` — state mutations
- `src/config.js` — safe public config and constants
- `styles.css` — styling

If adding a build step, prefer Vite and keep GitHub Pages deployment simple.
