# Codex Working Notes

This is a personal family dashboard for a non-developer owner. Prioritize reliability, clear documentation, and changes that can be maintained by future Codex sessions.

## Product Intent

- Primary device: Surface Pro in portrait orientation, running in kiosk mode.
- Primary use: walk-by household status at a glance.
- Secondary use: tap into chores, approvals, history, and management.
- Audience: family members, including kids.
- Tone: clear, calm, household-oriented. Avoid fake enterprise language in user-facing text.

## Engineering Rules

- Keep the app deployable on GitHub Pages.
- Do not add a backend unless the owner explicitly approves it.
- Prefer Firebase client SDK plus secure Realtime Database rules.
- Do not commit Firebase Admin SDK service account files or private keys.
- Firebase client config is allowed in the frontend. Security must come from rules and auth.
- Keep changes small and verifiable.
- After frontend edits, run the app locally and check browser console/runtime behavior.

## Current Shape

- `index.html` contains HTML, CSS, JavaScript, Firebase setup, default data, and rendering.
- Firebase state path: `familyDashboard/state`.
- Local fallback key: `fb_v11`.
- Google Calendar iframe is configured inside `GOOGLE_CAL_URL`.

## Near-Term Refactor Direction

Do not split files just for style. Split only when making a substantive change:

- `src/state.js` for Firebase/local state.
- `src/render.js` for UI rendering.
- `src/actions.js` for mutations.
- `src/config.js` for safe public config and constants.
- `styles.css` for styling.

If adding a build step, prefer Vite and keep GitHub Pages deployment simple.

