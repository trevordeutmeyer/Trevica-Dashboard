# Data Model

The current app stores one state object at:

```text
familyDashboard/state
```

## Phase 1 State Shape

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
  "weekKey": "w2026_23",
  "settings": {
    "dinnerEnabled": false,
    "dinnerMenu": ""
  },
  "dinnerEnabled": false,
  "dinnerMenu": "",
  "meta": {
    "createdAt": "2026-06-08T00:00:00.000Z",
    "createdAtMs": 1780876800000,
    "createdBy": "client_abc",
    "updatedAt": "2026-06-08T00:00:00.000Z",
    "updatedAtMs": 1780876800000,
    "updatedBy": "client_abc",
    "writeId": "write_abc",
    "reason": "chore.toggle"
  }
}
```

`logs` is kept as a compatibility alias for older local/Firebase data. New code writes structured history to `activity` and mirrors that array into `logs` while the app remains a single-file static page.

## Person

```json
{
  "id": "jules",
  "name": "Jules",
  "emoji": "star",
  "role": "child",
  "weeklyAllowance": 5,
  "streak": 3,
  "chores": []
}
```

## Chore

```json
{
  "id": "j1",
  "name": "Make bed",
  "type": "weekly",
  "amt": 0
}
```

`type` is currently either:

- `weekly`: core weekly chore.
- `job`: optional paid job that requires parent approval.

## Pending Approval

```json
{
  "id": "abc123",
  "type": "allowance",
  "personId": "jules",
  "amt": 5
}
```

Job approvals also include `choreId`.

## Activity

```json
{
  "id": "activity_abc",
  "type": "chore",
  "action": "chore.completed",
  "message": "Jules completed: Make bed",
  "msg": "Jules completed: Make bed",
  "time": "6/8/2026 7:15 AM",
  "createdAt": "2026-06-08T13:15:00.000Z",
  "createdAtMs": 1780924500000,
  "weekKey": "w2026_23",
  "clientId": "client_abc",
  "actorId": "jules",
  "personId": "jules",
  "targetType": "chore",
  "targetId": "j1"
}
```

The app keeps the newest 80 activity items. Supported Phase 1 actions include chore completion/reopen, shared task creation/completion, approval request/approve/deny/reopen, allowance changes, dinner updates, completed-task clearing, and weekly reset.

## Realtime Sync

On startup the app:

1. Loads `localStorage` key `fb_v11`, then normalizes it to the current schema.
2. Opens a Firebase Realtime Database listener at `familyDashboard/state`.
3. Seeds Firebase with the normalized local/default household only if the cloud path is empty.
4. Caches every remote snapshot back to `localStorage` without writing it back to Firebase.
5. Stamps local writes with `meta.updatedAt`, `meta.updatedBy`, `meta.writeId`, and a short `meta.reason`.

## Future Direction

The single shared state object is still simple, and every write overwrites the whole app state. That is acceptable for one kiosk and light household use, but it can cause conflicts if multiple phones/tablets edit at the same time. If multi-device use becomes common, split state into paths such as:

```text
familyDashboard/people
familyDashboard/chores
familyDashboard/weeks/{weekKey}
familyDashboard/pending
familyDashboard/logs
familyDashboard/settings
```

