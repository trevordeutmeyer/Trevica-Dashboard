# Operations

## Source of Truth

GitHub repo:

```
https://github.com/trevordeutmeyer/Trevica-Dashboard
```

Working copy (local):

```
C:\DeutmeyerOS\Trevica-Dashboard
```

Do not edit loose copies of `index.html` on the Desktop. That creates version drift.

## Deploying Changes

1. Edit `index.html` in the working copy.
2. Commit and push to `main`.
3. GitHub Pages deploys automatically (usually within 60 seconds).
4. The kiosk picks up the update automatically — see Auto-Update below.

## Auto-Update (Kiosk)

The app polls its own GitHub Pages URL via HTTP HEAD every 10 minutes and compares `ETag` / `Last-Modified` headers.

- If a new deployment is detected and the screen has been idle for more than 45 seconds, it reloads automatically.
- If someone is actively using the screen, a "tap to update" bar appears at the bottom.
- A hard reload also runs every night at 3 AM regardless.
- If you need an immediate reload, tap ⚙ → 🔄 Reload App.

## Surface Pro Kiosk Setup

Recommended browser: Microsoft Edge.

1. Open the GitHub Pages URL.
2. Set portrait orientation in Windows display settings.
3. Enable Windows Assigned Access / kiosk mode once the app is stable.
4. Disable sleep while plugged in (Settings → Power → Screen timeout = Never).
5. Keep screen timeout long for household use.
6. Bookmark the URL for family phone access.

## Local Development

```powershell
cd C:\DeutmeyerOS\Trevica-Dashboard
python -m http.server 8080
```

Open `http://localhost:8080/`. Do not open `index.html` directly from the file system — Firebase and some browser APIs behave differently on `file://` origins.

## Daily Use

| Task | How |
|---|---|
| Check chores | Tap any chore row on the home screen |
| Add a quick task | Tap the **+** button |
| Approve allowance / job | Tap the person card header → approve in modal |
| Schedule an announcement | ⚙ → Family Settings → Custom Banners |
| Set dinner plan | ⚙ → Family Settings → Show dinner plan |
| Review activity | ⚙ → Activity Log |
| Start new week | ⚙ → Family Settings → Start New Week (PIN required) |
| Reload app | ⚙ → Reload App |

## Parent PIN

Default PIN is `1234`. It is hard-coded in `index.html` as `PARENT_PIN`. It is a visual deterrent only — not real security. Change it by editing the constant directly until Firebase Authentication is implemented.

PIN is required for: Family Settings, Start New Week, adjusting allowances, deleting chores.

PIN is not required for: checking chores, adding tasks, approving items from the home screen.

## Recovery

If the screen looks wrong or shows stale data:

1. Tap ⚙ → Reload App.
2. Check the sync status dot in the header (green = Firebase live, amber = offline/local).
3. If offline, check internet connectivity on the Surface.
4. If data is stale on one device, close and reopen the browser.
5. If Firebase data is corrupted, export a backup from the Firebase Console and restore manually, or wipe `familyDashboard/state` and let the app re-seed from defaults.

## Theme Behavior

- Light theme: 7 AM to 8 PM.
- Dark theme: 8 PM to 7 AM.
- Switching happens on page load. A reload is needed for an immediate switch outside the normal hour boundary.
- The inline `<script>` in `<head>` sets `data-theme` before any rendering to prevent a flash.
