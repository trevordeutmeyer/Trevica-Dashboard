# Trevica Dashboard

Personal family dashboard for the Deutmeyer household. The app is designed to run on a Surface Pro in portrait kiosk mode and provide a quick walk-by view of chores, shared tasks, approvals, dinner, history, and calendar events.

## Live App

GitHub Pages URL:

https://trevordeutmeyer.github.io/Trevica-Dashboard/

Repository:

https://github.com/trevordeutmeyer/Trevica-Dashboard

## Current Architecture

- Static GitHub Pages app.
- Main application file: `index.html`.
- Firebase Realtime Database stores shared dashboard state at `familyDashboard/state`.
- Browser `localStorage` is used as a fallback when Firebase is unavailable.
- Google Calendar is embedded in an iframe.

## Local Work

From the repo folder:

```powershell
python -m http.server 8080
```

Then open:

```text
http://localhost:8080/
```

If Python is not available, the app can also be opened directly from `index.html`, but a local server is closer to how GitHub Pages behaves.

## Priorities

1. Stabilize the current app so the kiosk never opens to a broken screen.
2. Lock down Firebase rules before the dashboard becomes depended on.
3. Replace the hard-coded parent PIN with real Firebase Authentication.
4. Improve portrait layout for the Surface Pro kiosk.
5. Split the single HTML file into maintainable files once behavior is stable.

## Important Security Note

The Firebase web config in `index.html` is public app configuration. That is expected for Firebase web apps. The real security boundary is Firebase Authentication plus Realtime Database Security Rules.

The current parent PIN is only a visual deterrent because it is shipped in client-side code. It should not be treated as real security.

