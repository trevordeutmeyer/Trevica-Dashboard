# The Deutmeyer Family Dashboard — Master Specification

## 1. What It Is

The Deutmeyer Family Dashboard is a permanent, always-on family command center built as a single HTML web application running 24/7 on a table-mounted Surface Pro. It is the single source of truth for the household: chores, allowances, calendar, tasks, announcements, and history — all visible at a glance from across the room.

It syncs in real time across every family device via Firebase and is hosted on GitHub Pages so any phone, tablet, or laptop can access it via a bookmarked URL.

## 2. Display and Viewing Requirements

- The primary display is a Surface Pro in portrait orientation on a side table.
- Text, buttons, checkboxes, and metrics are sized for readability and tapping from 3–6 feet away.
- Layout is touch-first with large tap targets.
- The app is also responsive across phone portrait, phone landscape, tablet portrait, and desktop landscape — see Section 3.
- Theme switches automatically between light (7 AM–8 PM) and dark (8 PM–7 AM) using a no-flash inline script in `<head>`.

## 3. Responsive Layout

The app detects screen size and orientation and adjusts layout automatically.

**Surface Pro portrait / tablet portrait (768–1099 px wide)**
Default kiosk layout. Fixed 100vh, no scroll. From top to bottom:
- Manga header (clock + title)
- Smart announcement banner
- Calendar strip (fixed height)
- Kids hero grid (2-column, fills remaining space)
- Adults compact strip (fixed height)

**Desktop / tablet landscape (≥ 1100 px wide)**
Two-column CSS grid. Left column (≈360 px): stats, calendar, adults. Right column: kids hero grid at full height.

**Phone portrait (< 600 px wide)**
Single column, natural scrolling. Kids cards stacked vertically. Calendar and adults shrink. Header and text scaled down.

**Phone landscape (height < 520 px)**
Compact two-column scrollable layout. Calendar and adults on the left; kids on the right. Header strips to essentials.

## 4. Visual Design

**Header**
Manga street-art style panel. "DEUTMEYER" in large outlined amber type with speed lines and a halftone dot texture. "DASHBOARD" in a parallelogram ink tag. Animated star starburst badge. Live clock (large) and date on the right. Sync status dot and label below the title.

**Color system**
Warm amber accent (`#d97706`). Light theme: warm off-white backgrounds (`#f7f4ef`). Dark theme: near-black (`#100e0c`). Family member accent colors: Dad = green, Mom = red/coral, Jules = sky blue, Henry = orange.

**Typography**
Plus Jakarta Sans, 600/700/800 weights. All body text scaled up for walk-by readability. Kid card names at 26 px, chore rows at 16 px, checkboxes at 30×30 px.

## 5. Screen Layout — Home Panel

**Smart announcement banner** — full-width strip below the header. See Section 9 for priority logic.

**Stats strip** — three mini cards: total chores done/total, pending approvals, current week number.

**Kids hero grid** — Jules and Henry occupy center stage in equal-height side-by-side cards. Each card shows: emoji, name, streak tag, progress bar, and all weekly chores as tappable rows. Tapping a row toggles it. Tapping the card header opens the full person detail modal.

**Calendar strip** — Google Calendar embedded iframe, full width.

**Adults compact strip** — Dad and Mom side by side in a compact horizontal row. Shows task list; tapping a row toggles it. Tapping the card opens the person detail modal.

## 6. Kids' Chore and Allowance System

**Weekly allowance — all or nothing**
Each child has a weekly allowance amount set by a parent. All core weekly chores must be completed to unlock allowance. Completing all chores auto-submits an approval request. After parent approval, the allowance is paid. If a chore is unchecked after approval, the approval is revoked and the chore list resets.

**One-time paid jobs**
Custom jobs with a dollar amount, added via the + button. No PIN required to add. Completing a job submits an individual approval request. Jobs are cleared on new-week reset.

**Streak tracking**
Consecutive weeks of fully completed and approved chores. Displayed as a badge on each kid card. Resets to zero if a week is missed.

## 7. Adult Task Lists

Dad and Mom each have task cards. Tasks are simple checklist items — no dollar amounts, approval flow, or streaks. Tasks reset on new-week reset.

## 8. Person Detail Modal

Tapping any person card header opens a modal showing:
- Name, emoji, progress bar
- All chores as tappable rows (tap to check or uncheck)
- For parents: pending approval items with approve/deny buttons

## 9. Smart Announcement Banner

Priority order, highest first:

1. **Movie-time alert** (hard-coded): 7:35–8:05 PM shows a live countdown to Family Movie Time with a red pulsing alert style.
2. **Custom scheduled banners**: any banner from `state.customBanners[]` whose current time falls within its start/end window. Most recently started banner wins if multiple are active. Styles: Note (dark tag), Event (amber tag), Alert (red tag + pulsing).
3. **Dinner menu**: if enabled by a parent in settings and a menu is set.
4. **Rotating welcome messages**: 7 kid-friendly messages selected by `(day + hour) % 7`, cycling through the day.

## 10. Custom Banner Scheduling

Parents can schedule banners in Family Settings (PIN required). Each banner has:
- Message text
- Start datetime
- End datetime
- Style: Note / Event / Alert

The settings panel shows all scheduled banners with a LIVE indicator for currently active ones. Past banners are shown dimmed. Banners persist in Firebase and sync across devices.

## 11. Settings and Navigation

Navigation is accessed via the ⚙ gear icon in the top-right of the header. Tapping it opens a small popup with:
- 📋 Activity Log
- ⚙️ Family Settings (PIN required)
- 🔄 Reload App

The + floating action button always visible for quick task creation.

## 12. Family Settings (PIN-gated)

- Dashboard settings: dinner plan toggle and text
- Custom banners: schedule, list, and delete
- Per-person: adjust weekly allowance, add/delete chores
- Start New Week (destructive reset, also PIN-gated)

## 13. Approvals System

Pending requests are surfaced inline at the top of each parent's card on the home screen, and in full detail in the person modal. Each request shows who is requesting, what they completed, and the dollar amount. Parents approve or deny individually.

## 14. History and Activity Log

Accessible from the settings popup. Timestamped log of all chore completions, approvals, denials, task additions, and week resets. Append-only, capped at 80 entries. Persists in Firebase.

## 15. Kiosk Auto-Update

An IIFE runs on page load and:
- Polls the GitHub Pages URL via HEAD request every 10 minutes, comparing `ETag` / `Last-Modified` headers.
- If a new deployment is detected and the screen has been idle > 45 s, reloads automatically after 1.8 s.
- If the screen is active, shows a tap-to-update bar at the bottom.
- Schedules a nightly hard reload at 3 AM.
- Also polls on `visibilitychange` (Surface wake from sleep).

## 16. Google Calendar Integration

Google Family Calendar embedded as an iframe in the calendar strip on the home panel. A CSS filter (`invert + hue-rotate`) adapts it to the dark theme automatically.

## 17. Gamification

- Tapping a chore checkbox plays a Web Audio API oscillator chime.
- Completing a single chore fires a small confetti burst.
- Completing all weekly chores triggers a full-screen confetti explosion.
- Parent approval plays a distinct cash-register chime.
- Kid name/emoji/streak tags are large and expressive.

## 18. Firebase Real-Time Sync

All state lives in Firebase Realtime Database at `familyDashboard/state`. Every action syncs instantly across family devices. `localStorage` (`fb_v11`) is the offline fallback. See `docs/FIREBASE.md` for sync behavior and security guidance.

## 19. New Week Reset

Triggered manually (PIN) or automatically on page load when the calendar week changes. Clears: weekly chore checkmarks, earned totals, one-time jobs. Keeps: weekly chore definitions, allowance amounts, streaks (updated based on prior week completion), activity log.

## 20. Hosting

- GitHub Pages, `main` branch root, no build step.
- Primary URL: `https://trevordeutmeyer.github.io/Trevica-Dashboard/`
- Surface Pro runs it full-screen in Edge kiosk mode.
- Any family device can access via bookmarked URL.
