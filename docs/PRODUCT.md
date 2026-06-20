# Product Notes

## Goal

A family command center that is useful at a glance from across the room and easy to manage from any device.

## Main Kiosk View — Questions It Answers

- Who has completed their chores today?
- What tasks are still open?
- Are any approvals waiting?
- What is for dinner?
- What is on the calendar this week?
- Is there a family announcement or event today?

## Core Workflows

- Walk by and read chore status at a glance.
- Tap a chore row to check or uncheck it.
- Tap a person card header to open the full detail modal.
- Parent approves or denies allowance/job requests from the person modal or the home card.
- Add a quick shared task via the + button.
- Add or manage custom banners via Family Settings.
- Start a new weekly loop from Family Settings.
- Update dinner plan from Family Settings.
- Review recent history from the settings popup.

## Visual Identity

- **Header**: Manga street-art style — "DEUTMEYER" in large outlined amber type with SVG speed lines, halftone dot texture, animated starburst badge, ghost FX word.
- **Accent color**: Warm amber (`#d97706`).
- **Theme**: Automatically light (7 AM–8 PM) and dark (8 PM–7 AM).
- **Typography**: Plus Jakarta Sans. Body copy and chore rows are large by web standards — designed for walk-by readability, not desktop density.
- **Person colors**: Dad = green, Mom = coral, Jules = sky blue, Henry = orange.
- **Person emojis**: Henry = ⚽, Jules = 🐎, Mom = 🌱 (gardening), Dad = 🪚 (carpentry).

## Design Direction

- Portrait-first on the Surface Pro kiosk; responsive for all other screens.
- Large touch targets (chore rows min 64 px tall, checkboxes 30×30 px).
- Walk-by readability: kid names at 26 px, chore text at 16 px.
- No tabs or dense navigation on the first screen. Settings one tap deeper via gear icon.
- Gamification for kids: confetti, chimes, streaks, expressive emoji badges.
- Announcements feel like manga panel captions (parallelogram tags, bold labels).

## Chore Day Scheduling

Each chore can optionally be assigned to specific days of the week (e.g., "take out trash" on Mon/Thu). Day-specific chores:

- Show a small day badge next to the chore name (Mon, Tue, etc.)
- Today's chores have an amber-filled badge
- Overdue chores (past due this week, not yet done) show a red badge and a faint red row tint
- Future-due chores show a muted badge
- Chores are sorted: today-due first, then overdue, then future, then undone unscheduled, then completed

Chores without day assignments are "unscheduled weekly" — they reset at the start of each week and can be done any day.

## Chore Reset and Tracking

Chore completions are date-stamped (`YYYY-MM-DD`), not stored as a flat boolean. This means:

- **Day-specific chores** reset automatically each day — checked off Monday, they reappear Tuesday.
- **Unscheduled weekly chores** reset each Monday — checked off any time this week, they stay checked until the new week.
- A chore checked off earlier in the week still counts toward the weekly allowance progress bar even after it resets.

This eliminates the need for a manual "start new week" to clear chores — daily chores self-manage.

## Motion Wake (Camera-Based Sleep Screen)

When enabled in Family Settings, the kiosk uses the front-facing camera to detect motion:

- **Idle**: if no motion is detected for 2 minutes, the screen dims to a minimal sleep view (clock + date only).
- **Wake**: motion above a configurable pixel-diff threshold wakes the screen instantly.
- No images or video are stored or transmitted — only per-frame brightness diffs are computed.
- Requires browser camera permission (prompted on first enable).
- Also uses the Wake Lock API to prevent the OS from sleeping the browser.
- Toggle is in Family Settings (PIN not required to toggle, PIN required to enter settings).

## Banner System

Banners are the primary communication channel for time-sensitive family messages. Priority:
1. Movie-time alert (automatic, 7:35–8:05 PM)
2. Scheduled custom banners (parent-created, date/time windowed)
3. Dinner plan (if set)
4. Rotating welcome messages (7 kid-friendly phrases, cycle by time of day)

Custom banners allow parents to schedule messages for specific occasions — school events, sports days, holiday reminders, etc. — that automatically appear and disappear without manual intervention.

## Open Decisions

- Whether parents need separate authenticated logins vs. shared PIN.
- Whether kids should have limited phone access (read-only view vs. full app).
- Whether calendar should move to a custom Firebase event list instead of Google Calendar embed.
- Whether streak tracking should show a historical streak graph.
- Whether to add push notifications for approval requests to parent phones.
- Whether to add a kid-facing phone view (read-only chore list + allowance balance).
- Whether to add earned/paid allowance balance tracking (Greenlight-style running total).
