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
  Fields: `id` (ms timestamp), `plate`, `vno` (property no.), `desc`, `who` (assigned person), `repair` (Preventive / Corrective / Express Maintenance), `status` (`bay`, `wait` or `sched`), `since` (ms timestamp, reset when moved between columns), `limitHrs`, `bay` (text such as "Bay 1"), `mechanics` (array, max 5 names).
  `work` (work description text, shown on the customer board), `dateIn` ('YYYY-MM-DD', date the vehicle entered the bay area; set on add, defaults to today, editable in Staff panel, never reset by dragging). `remarks` (free text such as "Waiting for parts", shown on the customer board under Work, editable in Staff panel). `schedDate` ('YYYY-MM-DD') and `schedTime` ('HH:MM', optional) for `sched` entries; a scheduled entry has `dateIn` '' until it is moved to Waiting/In bay (drag or Edit), which sets `dateIn` to today and resets `since`. Entries without `dateIn` fall back to the date of `id` (or `since` for small demo ids).
- Doc `meta/roster` with `{ names: [...] }`, the mechanics list used by the pickers.
- Collection `history`, one doc per finished job, doc id = `String(entry.id)`: all entry fields plus `doneAt` (ms), `bayMs` (ms in bay = doneAt - since), `overLimit` (bool). Written by the Staff panel "Done" button (in-bay vehicles only) in one batch with deleting the `entries` doc. Read only while the Finished page is open (`subscribeHistory`, ordered by `doneAt` desc). Allowed by the existing `{document=**}` rule, so no rules change was needed.
- `VEHICLES` (127 items: `no`, `plate`, `desc`, `who`) is embedded in `index.html`. It powers the searchable vehicle field. Plate `N/A` means no plate, so the board shows the property number as the main label. The assigned person's name is shown in the Staff panel only, not on the customer display.

## Behaviors to preserve
- Board layout: In the bay now (left); Waiting, then Scheduled (right column). Waiting rows show only Date in (the waiting timer was removed at the user's request). Scheduled rows are sorted by date/time and labelled Today (amber), Tomorrow, weekday date, or Overdue (red).
- Staff panel has three columns: Scheduled, Waiting, In bay. Each row has one-click move buttons (Scheduled: "Arrived → Waiting" / "Arrived → In bay"; Waiting: "Move to In bay"; In bay: "Move to Waiting") because dragging was hard and does not work on touch screens. Dragging still works on computers. Both go through `moveTo()`.
- Vehicles can also be dragged directly on the main board (not in TV mode, mouse only): drop zones `#bayZone`, `#queueZone`, `#schedZone`. While dragging, `boardDragging` pauses the 1-second `renderBoard()` redraw so the dragged card isn't replaced. Moving into Scheduled (board or Staff drag) asks for the date with a prompt (default tomorrow); cancelling leaves the vehicle where it was.
- Elapsed time ticks every second; the card turns red with "OVER LIMIT" past the limit.
- Staff panel has two columns (Waiting, In bay); vehicles are dragged between them. Dragging resets the timer (`since`).
- Every field is editable at any time, including while waiting, via the **Edit** button on each vehicle in the Staff panel (In bay and Waiting). Edit loads the vehicle into the top form (title "Edit …", "Save changes" / "Cancel edit"); it can change the vehicle (duplicate guard applies), repair type, work, remarks, date in, status, bay, mechanics, time limit and the timer start (`fSince`). Changing status without touching the timer start resets `since`, like dragging. The Staff rows show a read-only summary; the old inline editors were replaced.
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

## Finished maintenance page
- `#finished` in the URL (or the "Finished" button) shows the page instead of the board; "Back to board" clears the hash. Hidden in TV mode.
- Filters: text search, finished from/to date, repair type. Summary line counts jobs, within/over limit, average time in bay.
- Per row: Undo (puts it back in the bay with its original `since`, blocked if the vehicle is already on the board) and Delete (with confirm).
- "Export to Excel (CSV)" downloads the filtered list (UTF-8 BOM so Excel reads it). The CSV includes the end-user; the on-screen table does not.
- "Remove" in the Staff panel still deletes without recording (now asks for confirmation); "Done" is the normal way to finish a job.
- Never name a module variable `history` — it would hide `window.history`, which TV mode uses.

## Testing
Serve the folder (`python3 -m http.server`) and open it in a browser with `FIREBASE_CONFIG.apiKey` empty to test in local mode. A headless-browser smoke test (add vehicles, check cards, duplicate guard) was used during development. Live Firestore sync has not been tested from the development sandbox, which cannot reach Google.

## Status and ideas
- As of 2026-11-05: files delivered to the user; they were creating the GitHub repo and turning on Pages.
- 2026-11-05: the upload landed as `index (1).html` (and `index.html` was deleted), which Pages can't serve. Renamed back to `index.html`, added README/CLAUDE.md/firestore.rules to the repo, and changed README so it no longer tells the user to paste `firestore.rules` into Firebase (that would have replaced the bid monitoring site's rules).
- 2026-11-05: user renamed the file on main themselves; site is live. TV display mode added (tested headless: alert bar, near/over badges, one chime per crossing, auto-scroll).
- 2026-11-05: added work description and date-in fields (form, Staff panel editors, bay cards and waiting rows).
- 2026-11-05: added Remarks field (same places as work description).
- 2026-11-05: added Done button and Finished maintenance page (tested headless: done, filters, CSV export, undo, delete, hidden in TV mode).
- 2026-11-05: added Edit button for In bay and Waiting vehicles (tested headless: edit all fields, timer start, vehicle swap with duplicate guard, status change, cancel).
- 2026-11-05: removed the waiting timer from the board; added the Scheduled area (status `sched`).
- 2026-11-05: added one-click move buttons in the Staff panel (user found dragging hard).
- 2026-11-05: added drag-and-drop on the main board; Date in on waiting rows no longer wraps; space at the bottom for the floating buttons.
- Ideas the user has not decided on: staff PIN, link to the user's PM/CM maintenance records.
