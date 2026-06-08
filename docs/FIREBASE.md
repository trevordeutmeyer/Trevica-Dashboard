# Firebase Notes

## Current Use

The app uses Firebase Realtime Database from client-side JavaScript.

Current database path:

```text
familyDashboard/state
```

Current Firebase app config is in `index.html`.

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

