# Trevica Dashboard

Personal family dashboard for the Deutmeyer household. Designed to run 24/7 on a Surface Pro in portrait kiosk mode — readable at a glance from across the room — and also usable on phones, tablets, and desktop browsers.

## Live App

```
https://trevordeutmeyer.github.io/Trevica-Dashboard/
```

Repository:

```
https://github.com/trevordeutmeyer/Trevica-Dashboard
```

## Current Architecture

- Single-file static GitHub Pages app (`index.html` contains all HTML, CSS, and JS).
- Firebase Realtime Database stores shared state at `familyDashboard/state`.
- `localStorage` key `fb_v11` is the offline fallback.
- Google Calendar is embedded via iframe.
- No build step. Edit `index.html`, push to `main`, GitHub Pages deploys automatically.

## Responsive Layout

The app detects the viewing environment and adjusts automatically:

| Context | Layout |
|---|---|
| Surface Pro portrait (768–1099 px) | Kids hero grid + calendar strip + adults strip, fixed 100vh |
| Desktop / tablet landscape (≥ 1100 px) | Two-column: calendar + adults on left, kids fill right |
| Phone portrait (< 600 px) | Single-column, scrollable |
| Phone landscape (height < 520 px) | Compact two-column, scrollable |

## Local Development

```powershell
python -m http.server 8080
```

Then open `http://localhost:8080/`. A local server is preferred over opening `index.html` directly because it avoids CORS quirks with Firebase.

## Key Files

| File | Purpose |
|---|---|
| `index.html` | Entire application |
| `AGENTS.md` | Engineering rules for AI-assisted development |
| `MASTER_SPEC.md` | Full product specification |
| `docs/DATA_MODEL.md` | Firebase/localStorage state schema |
| `docs/FIREBASE.md` | Firebase sync behavior and security notes |
| `docs/OPERATIONS.md` | Kiosk setup, daily use, recovery |
| `docs/PRODUCT.md` | Design intent and open decisions |

## Priorities

1. Keep the kiosk stable — the Surface Pro should never open to a broken screen.
2. Lock down Firebase rules and add authentication before treating the app as secure.
3. Replace the hard-coded parent PIN (`1234`) with real Firebase Authentication.
4. Split `index.html` into maintainable source files once behavior is stable.

## Security Note

The Firebase web config committed to this repo is public app configuration — that is normal for Firebase web apps. Real security comes from Firebase Authentication and Realtime Database Security Rules, neither of which is fully configured yet. Do not store sensitive family data here until auth rules are in place.
