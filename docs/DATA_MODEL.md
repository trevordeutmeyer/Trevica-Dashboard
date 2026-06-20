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
  "weekKey": "2026-06-15",
  "settings": {
    "dinnerEnabled": false,
    "dinnerMenu": "",
    "motionWake": false
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

### weekKey format

`weekKey` is the ISO date string of the **Monday** of the current week, e.g. `"2026-06-15"`. It is recalculated from `_mondayOf(new Date())` on every page load — the stored value is replaced, never trusted. This guarantees consistent week boundaries regardless of what was previously saved.

> **Previous format** (before June 2026): `"w2026_25"` — a formula-based string that had a time-of-day dependency causing Mon–Fri completions to appear in a different week than Sat–Sun. No longer used.

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
| `henry` | Henry | ⚽ | child | 7.00 |
| `jules` | Jules | 🐎 | child | 5.00 |
| `mom` | Mom | 🌱 | parent | 0 |
| `dad` | Dad | 🪚 | parent | 0 |

`role` is either `"parent"` or `"child"`. Parents do not have allowance, streaks, or approval flow.

## Chore

```json
{
  "id": "j1",
  "name": "Make bed",
  "type": "weekly",
  "amt": 0,
  "days": [1, 3, 5]
}
```

`type` values:
- `"weekly"` — core weekly chore, required for allowance unlock.
- `"job"` — one-time paid job, requires individual parent approval.

`days` (optional array of integers, 0 = Sunday … 6 = Saturday): the specific days of the week this chore is due. When present and non-empty the chore is "day-specific" and resets each day. When absent or empty the chore is "unscheduled weekly" and resets each week.

Day-specific chores show a colored badge next to their name:
- **Today** — filled amber badge
- **Overdue** (past due this week, not yet done) — red tinted badge + row background tint
- **Future** — muted badge

## checked Schema

`state.checked` is a dict keyed by `personId`. Each value is a **date-stamped dict** (not an array):

```json
{
  "jules": {
    "j1": "2026-06-15",
    "j2": "2026-06-17"
  },
  "henry": {
    "h1": "2026-06-16"
  }
}
```

The value for each chore is the `YYYY-MM-DD` string of the date it was last checked off.

### isChecked logic

```
if chore.days is non-empty:
    done = (stored date === today)                            // resets each day
else:
    done = (dateToWeekKey(stored date) === weekKey)           // resets each week
```

### wasCheckedThisWeek logic

```
done_this_week = (dateToWeekKey(stored date) === weekKey)
```

Used for allowance progress — a day-specific chore checked on Monday still counts toward the weekly allowance on Saturday.

### Migration from old array format

`normalizeChecked` detects the old `[cid, cid, ...]` array format and migrates it by stamping each entry with the Monday of the current week. Migrated checks count as "done this week" on first load.

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
  "weekKey": "2026-06-08",
  "clientId": "client_abc",
  "actorId": "jules",
  "personId": "jules",
  "targetType": "chore",
  "targetId": "j1"
}
```

The log is capped at 80 entries (newest kept). Supported actions include: `chore.completed`, `chore.reopened`, `allowance.requested`, `allowance.approved`, `allowance.denied`, `job.completed`, `job.approved`, `job.denied`, `settings.dinner.updated`, `week.reset`, `banner.add`, `banner.delete`.

## Settings

```json
{
  "dinnerEnabled": false,
  "dinnerMenu": "",
  "motionWake": false
}
```

| Field | Type | Default | Description |
|---|---|---|---|
| `dinnerEnabled` | boolean | `false` | Whether to show the dinner plan banner |
| `dinnerMenu` | string | `""` | Dinner description text |
| `motionWake` | boolean | `false` | Camera-based motion wake: sleep screen when idle, wake on motion |

## State Normalization Pipeline

Every state load passes through:

1. `normalizeSettings(s)` — extracts `dinnerEnabled`, `dinnerMenu`, `motionWake`.
2. `normalizeHousehold(s.household)` — fills missing household fields.
3. `normalizePerson(p, index)` — fills missing person fields, applies migrations.
4. `normalizeChecked(s.checked)` — migrates old array format to date-stamped dict.
5. `normalizeState(input)` — assembles the full normalized object, runs one-time migrations. Always recalculates `weekKey` from `getWeekKey()`.

Migrations inside `normalizeState` (run only on existing saved data):
- `p.name === 'Jessica'` → `p.name = 'Mom'` (id: mom)
- `p.emoji === '⭐'` (Jules) → `p.emoji = '🐎'`
- `p.emoji === '🚀'` (Henry) → `p.emoji = '⚽'`
- `!streakResetDone && role === 'child'` → `streak = 0`

New fields added to state must be added to both `freshState()` and the `normalizeState` return object. New settings fields must also be added to `normalizeSettings`.

## Week Key Helpers

```javascript
function _mondayOf(d)           // returns 'YYYY-MM-DD' of the Monday of d's week
function getWeekKey()           // Monday of the current week
function getWeekMondayStr()     // alias of getWeekKey()
function dateToWeekKey(dateStr) // Monday of the week containing dateStr
```

All four use `_mondayOf`. ISO week: Monday = start, Sunday = end. A completion on any day Mon–Sun maps to the same weekKey as every other day in that week.

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
