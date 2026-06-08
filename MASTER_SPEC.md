# The Deutmeyer Family Dashboard - Master Specification

## 1. What It Is

The Deutmeyer Family Dashboard is a permanent, always-on family command center built as a single HTML web application running 24/7 on a table-mounted Surface Pro. It is designed to be the single source of truth for the entire household: chores, allowances, calendar, tasks, announcements, and history, all visible at a glance from across the room without ever needing to scroll, refresh, or dig through menus.

It runs full-screen in a browser, syncs in real time across every family device via Firebase, and is hosted on GitHub Pages so any phone, tablet, or laptop in the house can pull it up instantly via a bookmarked URL.

## 2. Physical Display And Viewing Requirements

- The entire main dashboard fits inside `100vh`, with no vertical scrolling on the main panels.
- Text, buttons, checkboxes, and metrics are much larger than a standard web app so they are readable and tappable from 3-6 feet away.
- The primary display is a Surface Pro in portrait orientation on a side table.
- The layout is touch-first, with large targets for tapping in motion.
- The app uses a dark premium theme throughout, with no white backgrounds in native UI.

## 3. Screen Layout And Visual Hierarchy

The screen is divided into fixed zones from top to bottom, all within one viewport:

1. Header bar: family name, live clock, live date.
2. Smart announcement banner: full-width family reminders, dinner plans, and automated Movie Time alert.
3. Navigation tabs: This Week, Approvals with live badge count, Calendar, History, Manage.
4. Main content area:
   - Kids first: Jules and Henry's chore cards occupy the most prominent center-screen space.
   - Adults below: Dad and Jessica's task cards sit beneath the kids' cards.
   - Shared family to-do pool: a dedicated centralized list visible to and editable by the whole family.

## 4. Kids' Chore And Allowance System

### Weekly Allowance: All Or Nothing

- Jules and Henry each have a weekly allowance amount set by a parent.
- They each have weekly core chores that must all be completed to unlock that allowance.
- Partial completion earns nothing.
- Once all chores are checked, the system automatically submits a payout request to Approvals.
- A child cannot receive allowance until a parent approves it.
- After approval, weekly chores lock until the next week reset.
- If denied, all chores uncheck and the child must complete them again.

### One-Time Jobs

- Parents or kids can add custom one-time jobs with a dollar amount at any time through the floating add button.
- No PIN is required to add one-time jobs.
- When a child marks a job complete, it submits for individual parent approval.
- Jobs do not bundle into weekly allowance.
- Approved jobs add to the child's earned total for the week independently.
- One-time jobs are removed automatically at the start of each new week.

### Streak Tracking

- Jules and Henry have a bold flame badge next to their names.
- The badge displays consecutive weeks in which all core weekly chores were completed and approved.
- The streak increments only after parent approval.
- A broken week resets the streak to zero.

## 5. Adult Task Lists

- Dad and Jessica each have their own task cards below the kids' cards.
- Adult tasks are simple to-do checklist items.
- Adult tasks do not use dollar amounts, approval flow, or streak tracking.
- Tasks can be added through the floating add button without a PIN.
- Adult tasks reset with the new week.

## 6. Shared Family To-Do Pool

- A centralized shared list exists for whole-house tasks.
- Any family member can add to it, check items off, or clear it without a PIN.
- It is visible on the main dashboard as its own section.
- Items persist until manually cleared and do not auto-reset on new week.

## 7. Floating Task Creation

- A floating add button is always visible as an overlay on the main screen.
- Tapping it opens a full-screen blurry glass overlay where family members can:
  - Type the task name.
  - Assign it to Dad, Jessica, Jules, Henry, or Shared.
  - Set a dollar amount for one-time kid jobs.
  - Select weekly chore or one-time job.
- No PIN is required to add tasks.
- PIN is only required for sensitive destructive actions.

## 8. Gamification For Kids

- Arcade checkboxes scale and/or rotate when tapped.
- Completing any individual chore fires a small confetti pop at the touch point.
- Completing all core weekly chores triggers a full-screen confetti explosion.
- Every chore checkbox tap plays a bright oscillator-based arcade chime.
- Parent approval of any payout plays a distinct cash-register-style oscillator sweep chime.
- Streak flame badges are prominent on kid cards.

## 9. Smart Announcement Banner

- A full-width notification strip sits directly beneath the header.
- Default state displays family reminders or the shared dinner plan, editable by parents.
- Every night from 7:35 PM to 8:05 PM, it overrides to display exactly:

  `"Family Movie Time begins in X minutes! Assemble at couch."`

- `X` is the live countdown in minutes from current time to 8:05 PM.
- At 8:05 PM, the banner automatically returns to default state.

## 10. Google Calendar Integration

- The Calendar tab displays the embedded Google Family Calendar.
- The main dashboard condenses the calendar into an ultra-minimalist full-width horizon strip showing the next few high-glance days.
- A CSS invert filter is applied to the embedded Google Calendar iframe to make it blend into the dark theme.
- The full calendar is accessible in the Calendar tab.

## 11. Parent PIN Security

- A 4-digit numeric PIN, `1234` by default, gates sensitive actions:
  - Starting a new week or resetting all chores.
  - Adjusting allowance amounts.
  - Deleting core weekly chores.
  - Undoing a previously approved payout.
- PIN entry uses an on-screen numeric keypad overlay.
- The Manage tab requires PIN entry.
- Adding tasks, checking chores, and clearing the shared list do not require a PIN.

## 12. Approvals System

- A dedicated Approvals tab has a live red badge showing pending request count.
- Every pending request displays who is requesting, what they completed, and the dollar amount.
- Parents can approve or deny each request individually.
- Approving a weekly allowance plays the cash-register chime and locks the kid's weekly chores.
- Denying a weekly allowance unchecks all that child's chores and removes the pending request.
- Approving a job pays it out and marks it as paid on the dashboard.
- Denying a job unchecks it and removes the pending request.

## 13. History And Activity Log

- A dedicated History tab maintains a timestamped activity trail of household activity.
- Every chore check, job completion, approval, and denial logs:
  - Person name.
  - Task name.
  - Action taken.
  - Exact date and time.
- The log is append-only and never deleted automatically.
- It persists in Firebase across devices.

## 14. Firebase Real-Time Sync

- The app connects to Firebase Realtime Database using official Firebase compat script links.
- Every action syncs instantly across family devices without refresh.
- State lives in Firebase, including chores, allowances, streaks, history, shared tasks, pending approvals, settings, and week metadata.
- Local storage is only a fallback.

## 15. Screensaver And Burn-In Protection

- From 8:00 PM to 7:00 AM, the app automatically enters dim standby screensaver mode.
- The screen dims significantly to protect the Surface Pro display.
- The screensaver may display a minimal clock or remain near-black.
- At 7:00 AM, the app restores to full brightness automatically.

## 16. New Week Reset

- The app automatically detects a new calendar week on page load and resets:
  - Weekly chore checkmarks clear.
  - Earned totals reset to zero.
  - One-time jobs are removed.
  - Weekly chores remain on the list.
  - Streaks update based on whether the prior week was fully approved.
- A manual PIN-protected New Week button also exists.

## 17. Hosting And Accessibility

- The app is hosted on GitHub Pages at a permanent family URL.
- Any device can access it by bookmarked URL.
- The Surface Pro runs it full-screen in Edge kiosk mode.
- Family phones and tablets can use the same URL to add tasks, check status, or view approvals remotely.

