# Firebase Notes

## Current Use

The app uses Firebase Realtime Database from client-side JavaScript.

Current database path:

```text
familyDashboard/state
```

Current Firebase app config is in `index.html`.

## Phase 1 Sync Behavior

The app remains a static GitHub Pages single-page app. It uses the Firebase client SDK only; there is no server or Firebase Admin SDK.

Startup flow:

1. Read the local fallback state from `localStorage` key `fb_v11`.
2. Normalize local data into schema version `1`.
3. Attach `.info/connected` and `familyDashboard/state` listeners.
4. If `familyDashboard/state` is empty, seed it once with the normalized local/default Deutmeyer household state using a transaction.
5. For each remote snapshot, normalize it, render it, and cache it locally without echoing the snapshot back to Firebase.

Local writes are debounced and written back to `familyDashboard/state` with `meta.updatedAt`, `meta.updatedBy`, `meta.writeId`, and `meta.reason`.

## Default Household Seed

The default seed includes:

- `household`: Deutmeyer family metadata and America/Denver timezone.
- `people`: Dad, Jessica, Jules, and Henry with starter weekly chores and allowance values.
- `settings`: dinner widget state.
- `activity` and `logs`: empty history arrays, with `logs` retained for compatibility.
- `meta`: client/write tracking for sync diagnostics.

## What Is Safe To Commit

Safe in frontend code:

- `apiKey`
- `authDomain`
- `databaseURL`
- `projectId`
- `storageBucket`
- `messagingSenderId`
- `appId`

Never commit:

- Firebase Admin SDK service account JSON.
- Private keys.
- Server API secrets.
- Passwords.

## Required Security Work

Before relying on this outside casual home use, enable Firebase Authentication and restrict Realtime Database rules.

Recommended direction:

1. Enable Firebase Authentication.
2. Create parent account(s).
3. Create kiosk account.
4. Require authenticated users for reads and writes.
5. Later, separate parent-only actions from kid/kiosk actions.

## Temporary Rule Warning

Rules like this are unsafe:

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

Anyone who discovers the app can read or change the dashboard if rules are public.

## Target Rule Direction

After auth is added, rules should move toward this shape:

```json
{
  "rules": {
    "familyDashboard": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

That is only a first locked-down version. Parent-only controls need more detailed rules and user roles.

## Phase 1 Rules Note

Phase 1 still writes one shared state object. The recommended first production rule remains authenticated read/write access to `familyDashboard`, followed later by role-aware rules for parent-only actions.

