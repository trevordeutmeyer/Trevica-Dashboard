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

## Design Direction

- Portrait-first on the Surface Pro kiosk; responsive for all other screens.
- Large touch targets (chore rows min 64 px tall, checkboxes 30×30 px).
- Walk-by readability: kid names at 26 px, chore text at 16 px.
- No tabs or dense navigation on the first screen. Settings one tap deeper via gear icon.
- Gamification for kids: confetti, chimes, streaks, expressive emoji badges.
- Announcements feel like manga panel captions (parallelogram tags, bold labels).

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
