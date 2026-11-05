# Bay Status Board

Customer-facing board for the Equipment Management Division: vehicles in the bay with a running timer against a time limit, the mechanics on each job, and a waiting queue. Staff update it from the **Staff** button; every open screen updates live.

Files: `index.html` (the whole site), `firestore.rules` (database rules).

## Put it online (GitHub Pages + Firebase)

### 1. Firebase (shared live data) — already done
The board uses your existing Firebase project `bid-monitoring-f1308`, and its settings are already filled in inside `index.html`. Nothing to do here.

**Do not change the Firestore Rules.** Your bid monitoring site depends on the rules that are there now, and they already let the board read and write. `firestore.rules` in this repo is only a note of what the board needs; do not paste it into Firebase, or the bid monitoring site may stop working.

### 2. GitHub Pages
1. Open the repository on GitHub: `https://github.com/marioibay19-tech/BAY-AREA-TIMER`.
2. Make sure the site file is named exactly **`index.html`** (not `index (1).html`). GitHub Pages only opens a file with that exact name.
3. Click **Settings** (top bar of the repository) → **Pages** (left menu).
4. Under **Build and deployment**: Source = **Deploy from a branch**, Branch = **`main`**, folder = **`/ (root)`** → **Save**.
5. Wait 1–2 minutes and refresh the Pages screen. The link appears at the top: `https://marioibay19-tech.github.io/BAY-AREA-TIMER/`

Open that link on the lobby TV and on the staff computer. A change on one shows on the other within about a second.

### 3. First check
1. Open the link. The top of the page should **not** show a yellow banner. (A banner saying "Demo mode" or "Could not reach Firebase" means live sharing is not working.)
2. Click **Staff**, add one vehicle, and check that it appears on a second device (for example your phone) within a few seconds.
3. Remove the test vehicle.

## Lobby TV (TV mode)
1. On the TV's browser, open **https://marioibay19-tech.github.io/BAY-AREA-TIMER/?tv** (note the `?tv` at the end) and bookmark it. You can also click **TV mode** at the bottom right of the normal page.
2. Tap the screen once, or press any key on the remote or keyboard. This turns on full screen and the alert sound. Browsers don't allow sound or full screen until someone taps.
3. That's it. The board shows bigger text, hides the Staff button, and keeps the screen awake where the browser allows it.

What TV mode shows:
- **"15 MIN LEFT"** (amber) when a vehicle is within 15 minutes of its time limit.
- **"OVER LIMIT"** (red, pulsing) and a red bar across the top listing every vehicle that is over time.
- A short **chime** once, when a vehicle first goes over its limit.
- If there are more vehicles than fit on the screen, the page scrolls slowly down and back up.

To leave TV mode, move the mouse and click **Exit TV mode** (top right).

## Notes
- Until `apiKey` is filled in, the page runs in demo mode (data stays on that one device) and shows a banner saying so.
- The Staff button only hides the controls; with the open rules above, anyone who has the link could edit. A simple PIN on the Staff panel can be added if you want it.
- Timers use the clock of the device that moved the vehicle into the bay, so keep device clocks set to automatic time.
