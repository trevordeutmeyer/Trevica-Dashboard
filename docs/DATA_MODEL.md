# Data Model

The current app stores one state object at:

```text
familyDashboard/state
```

## Current State Shape

```json
{
  "people": [],
  "checked": {},
  "approved": {},
  "pending": [],
  "logs": [],
  "globalTodos": [],
  "weekKey": "w2026_23",
  "dinnerEnabled": false,
  "dinnerMenu": ""
}
```

## Person

```json
{
  "id": "jules",
  "name": "Jules",
  "emoji": "star",
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

## Future Direction

The single shared state object is simple, but every write overwrites the whole app state. That is acceptable for one kiosk, but it can cause conflicts if multiple phones/tablets edit at the same time. If multi-device use becomes common, split state into paths such as:

```text
familyDashboard/people
familyDashboard/chores
familyDashboard/weeks/{weekKey}
familyDashboard/pending
familyDashboard/logs
familyDashboard/settings
```

