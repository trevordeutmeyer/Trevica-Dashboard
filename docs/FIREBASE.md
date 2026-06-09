# Firebase Notes

## Current Use

Firebase Realtime Database, client-side JavaScript only. No Firebase Admin SDK, no server.

Database path:

```
familyDashboard/state
```

Firebase app config is in `index.html`.

## Sync Behavior

Startup flow:

1. Read `localStorage` key `fb_v11` and normalize to current schema.
2. Attach `.info/connected` and `familyDashboard/state` listeners.
3. If `familyDashboard/state` is empty, seed it once with normalized local/default state using a transaction.
4. On each remote snapshot: normalize, render, cache to `localStorage`. Do not echo back to Firebase.
5. Local writes debounce and write to `familyDashboard/state` with `meta.updatedAt`, `meta.updatedBy`, `meta.writeId`, and `meta.reason`.

## Default Household Seed

The default seed (`DEFAULT_PEOPLE` + `DEFAULT_HOUSEHOLD` in `index.html`) includes:

| Field | Value |
|---|---|
| Household | Deutmeyer family, America/Denver timezone |
| Dad | 👨, parent, no allowance |
| Mom | 👩, parent, no allowance (migrated from "Jessica") |
| Jules | 🐎, child, $5/week allowance, streak 0 |
| Henry | ⚽, child, $7/week allowance, streak 0 |
| Dinner widget | disabled |
| Custom banners | empty array |
| streakResetDone | true |

## What Is Safe to Commit

Safe in frontend code:
- `apiKey`, `authDomain`, `databaseURL`, `projectId`, `storageBucket`, `messagingSenderId`, `appId`

Never commit:
- Firebase Admin SDK service account JSON
- Private keys or secrets
- Passwords

## Security Status — Current

Rules are likely open (`".read": true, ".write": true`). This means anyone who finds the URL can read or overwrite the dashboard. Acceptable for a casual private household kiosk; not acceptable if family data becomes sensitive.

## Required Security Work

Before treating the app as secure:

1. Enable Firebase Authentication.
2. Create parent and kiosk user accounts.
3. Update Realtime Database rules to require authentication:

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

4. Later: add role-aware rules so only authenticated parents can write to `settings`, `customBanners`, and approval decisions.

## Conflict Behavior

The app writes the entire state object on every change (no partial updates). If two devices write simultaneously, the last write wins. This is acceptable for current household use. If conflicts become a problem, move to per-path writes and Firebase transactions on sensitive fields.
