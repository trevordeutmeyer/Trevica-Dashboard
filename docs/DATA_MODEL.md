# Data Model

The app stores one state object at:

```
familyDashboard/state
```

Local fallback: `localStorage` key `fb_v11`.

## Top-Level State Shape

```json
{
  "schemaVersion": 1,
  "household": {
    "id": "deutmeyer",
    "name": "Deutmeyer Family",
    "timezone": "America/Denver",
    "dashboardTitle": "Deutmeyer Family Dashboard",
    "primaryDevice": "Surface Pro kiosk"
  },
  "people": [],
  "checked": {},
  "approved": {},
  "pending": [],
  "activity": [],
  "logs": [],
  "globalTodos": [],
  "weekKey": "w2026_24",
  "settings": {
    "dinnerEnabled": false,
    "dinnerMenu": ""
  },
  "dinnerEnabled": false,
  "dinnerMenu": "",
  "customBanners": [],
  "streakResetDone": true,
  "meta": {
    "createdAt": "2026-06-08T00:00:00.000Z",
    "createdAtMs": 1749340800000,
    "createdBy": "client_abc",
    "updatedAt": "2026-06-08T00:00:00.000Z",
    "updatedAtMs": 1749340800000,
    "updatedBy": "client_abc",
    "writeId": "write_abc",
    "reason": "chore.toggle"
  }
}
```

`logs` is a compatibility alias for `activity`. New code writes to `activity` and mirrors it into `logs`.

`dinnerEnabled` and `dinnerMenu` are duplicated at top level for backwards compatibility with older snapshots. New code reads from `settings.*`.

`streakResetDone` is a one-time migration flag. When `false` (old data), `normalizeState` resets all child streaks to 0 and then sets it to `true`. New installs start with `true`.

## Person

```json
{
  "id": "jules",
  "name": "Jules",
  "emoji": "🐎",
  "role": "child",
  "weeklyAllowance": 5.00,
  "streak": 0,
  "chores": []
}
```

Current people:

| id | name | emoji | role | weeklyAllowance |
|---|---|---|---|---|
| `dad` | Dad | 👨 | parent | 0 |
| `mom` | Mom | 👩 | parent | 0 |
| `jules` | Jules | 🐎 | child | 5.00 |
| `henry` | Henry | ⚽ | child | 7.00 |

`role` is either `"parent"` or `"child"`. Parents do not have allowance, streaks, or approval flow.

## Chore

```json
{
  "id": "j1",
  "name": "Make bed",
  "type": "weekly",
  "amt": 0
}
```

`type` values:
- `"weekly"` — core weekly chore, required for allowance unlock.
- `"job"` — one-time paid job, requires individual parent approval.

## Custom Banner

```json
{
  "id": "b1749340800000",
  "message": "Soccer tournament today — go Henry!",
  "type": "accent",
  "start": "2026-06-14T08:00",
  "end": "2026-06-14T18:00"
}
```

`type` values:
- `"info"` — dark/neutral tag labeled NOTE
- `"accent"` — amber tag labeled EVENT
- `"danger"` — red tag labeled ALERT, with pulsing animation

`start` and `end` are `datetime-local` strings (local time, no timezone offset). The banner engine compares them against `Date.now()` using `new Date(str).getTime()`.

## Pending Approval

```json
{
  "id": "abc123",
  "type": "allowance",
  "personId": "jules",
  "amt": 5.00
}
```

Job approvals also include `"choreId"`.

## Global Todo

```json
{
  "id": "todo_abc",
  "text": "Buy paper towels",
  "done": false,
  "createdAt": "2026-06-08T13:00:00.000Z"
}
```

## Activity Entry

```json
{
  "id": "activity_abc",
  "type": "chore",
  "action": "chore.completed",
  "message": "Jules completed: Make bed",
  "msg": "Jules completed: Make bed",
  "time": "6/8/2026 7:15 AM",
  "createdAt": "2026-06-08T13:15:00.000Z",
  "createdAtMs": 1749367200000,
  "weekKey": "w2026_24",
  "clientId": "client_abc",
  "actorId": "jules",
  "personId": "jules",
  "targetType": "chore",
  "targetId": "j1"
}
```

The log is capped at 80 entries (newest kept). Supported actions include: `chore.completed`, `chore.reopened`, `allowance.requested`, `allowance.approved`, `allowance.denied`, `job.completed`, `job.approved`, `job.denied`, `settings.dinner.updated`, `week.reset`, `banner.add`, `banner.delete`.

## State Normalization Pipeline

Every state load passes through:

1. `normalizeSettings(s)` — extracts `dinnerEnabled`, `dinnerMenu`.
2. `normalizeHousehold(s.household)` — fills missing household fields.
3. `normalizePerson(p, index)` — fills missing person fields, applies migrations.
4. `normalizeState(input)` — assembles the full normalized object, runs one-time migrations.

Migrations inside `normalizeState` (run only on existing saved data):
- `p.name === 'Jessica'` → `p.name = 'Mom'` (id: mom)
- `p.emoji === '⭐'` (Jules) → `p.emoji = '🐎'`
- `p.emoji === '🚀'` (Henry) → `p.emoji = '⚽'`
- `!streakResetDone && role === 'child'` → `streak = 0`

New fields added to state must be added to both `freshState()` and the `normalizeState` return object.

## Realtime Sync

On startup:
1. Load `localStorage` key `fb_v11`, normalize to current schema.
2. Open Firebase listener at `familyDashboard/state`.
3. If cloud path is empty, seed once with normalized local/default state.
4. On each remote snapshot: normalize, render, cache to `localStorage` without echoing back.
5. Local writes are debounced and written to Firebase with `meta.updatedAt`, `meta.updatedBy`, `meta.writeId`, `meta.reason`.

## Future Direction

The single shared state object works for one kiosk with light household use. If multi-device concurrent editing becomes an issue, split into per-concern paths:

```
familyDashboard/people
familyDashboard/chores
familyDashboard/weeks/{weekKey}
familyDashboard/pending
familyDashboard/logs
familyDashboard/settings
familyDashboard/customBanners
```
