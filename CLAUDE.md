# Bay Status Board

Customer-facing status screen for the DPWH Equipment Management Division (Regional Office 1) vehicle maintenance bay, like the board in an auto shop waiting area. It shows vehicles currently in the bay with a running timer against a time limit, the mechanics on each job, and a waiting queue. Staff update it from a "Staff" button; every open screen updates live.

The user is not a developer. Explain Firebase and GitHub steps in plain, numbered steps and say exactly where to click.

## Files
- `index.html` is the whole site: HTML, CSS and one `<script type="module">`. There is no build step and no npm.
- `firestore.rules` is for reference only (see Firebase below).
- `README.md` has the setup steps for the user.

## Stack and hosting
- Static site on GitHub Pages (repo `marioibay19-tech/BAY-AREA-TIMER`, branch `main`, root folder). Live URL: https://marioibay19-tech.github.io/BAY-AREA-TIMER/
- Shared live data in Firebase Firestore, project `bid-monitoring-f1308` (the same project the user's bid monitoring site uses). The web config is already filled in at the top of the module script (`FIREBASE_CONFIG`). Firebase web config values are public by design.
- Do NOT edit the project's Firestore rules without asking. They already allow all reads and writes (`match /{document=**}`) and the bid monitoring site depends on them. Anyone with the link can edit the board; this was accepted. A staff PIN is a possible later addition.
- If `apiKey` in `FIREBASE_CONFIG` is empty, the page runs in local demo mode using localStorage and shows a banner.

## Data model (Firestore)
- Collection `entries`, one doc per vehicle on the board, doc id = `String(entry.id)`.
  Fields: `id` (ms timestamp), `plate`, `vno` (property no.), `desc`, `who` (assigned person), `repair` (Preventive / Corrective / Express Maintenance), `status` (`bay` or `wait`), `since` (ms timestamp, reset when moved between columns), `limitHrs`, `bay` (text such as "Bay 1"), `mechanics` (array, max 5 names).
- Doc `meta/roster` with `{ names: [...] }`, the mechanics list used by the pickers.
- `VEHICLES` (127 items: `no`, `plate`, `desc`, `who`) is embedded in `index.html`. It powers the searchable vehicle field. Plate `N/A` means no plate, so the board shows the property number as the main label. The assigned person's name is shown in the Staff panel only, not on the customer display.

## Behaviors to preserve
- Elapsed time ticks every second; the card turns red with "OVER LIMIT" past the limit.
- Staff panel has two columns (Waiting, In bay); vehicles are dragged between them. Dragging resets the timer (`since`).
- Mechanics, bay and time limit are editable at any time, including while waiting.
- Adding a vehicle already on the board (same property no.) is blocked.
- All user-supplied text goes through `esc()` before `innerHTML`, because the data is public and shared.
- The Staff panel is not rebuilt while someone is typing in it (`staffBusy()`), so live updates don't wipe input.
- Timers use the clock of the device that moved the vehicle, so device clocks should be on automatic time.

## TV display mode
- Turned on by `?tv` in the URL (kept on refresh) or the "TV mode" button; `body.tv` class drives the bigger CSS. "Exit TV mode" (top right, hidden with the cursor after 3s idle) turns it off.
- First tap/key enables full screen, the Web Audio chime and a screen wake lock (browsers require a user gesture). The board still shows without the tap.
- Red alert bar lists over-limit vehicles. Chime plays once per vehicle when it newly crosses the limit, not for vehicles already over when the page loads (`alerted` set).
- "N MIN LEFT" amber badge within `NEAR_MS` (15 min) of the limit; this shows in normal mode too.
- Auto-scroll (`tvScroll`) when content is taller than the screen: pause 8s, scroll, pause 8s, jump to top.

## Testing
Serve the folder (`python3 -m http.server`) and open it in a browser with `FIREBASE_CONFIG.apiKey` empty to test in local mode. A headless-browser smoke test (add vehicles, check cards, duplicate guard) was used during development. Live Firestore sync has not been tested from the development sandbox, which cannot reach Google.

## Status and ideas
- As of 2026-11-05: files delivered to the user; they were creating the GitHub repo and turning on Pages.
- 2026-11-05: the upload landed as `index (1).html` (and `index.html` was deleted), which Pages can't serve. Renamed back to `index.html`, added README/CLAUDE.md/firestore.rules to the repo, and changed README so it no longer tells the user to paste `firestore.rules` into Firebase (that would have replaced the bid monitoring site's rules).
- 2026-11-05: user renamed the file on main themselves; site is live. TV display mode added (tested headless: alert bar, near/over badges, one chime per crossing, auto-scroll).
- Ideas the user has not decided on: staff PIN, link to the user's PM/CM maintenance records.
