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
- Do NOT edit the project's Firestore rules without asking. They already allow all reads and writes (`match /{document=**}`) and the bid monitoring site depends on them. Anyone with the link can technically write to the database; the staff PIN (below) only keeps customers out of the UI. This was accepted.
- If `apiKey` in `FIREBASE_CONFIG` is empty, the page runs in local demo mode using localStorage and shows a banner.

## Data model (Firestore)
- Collection `entries`, one doc per vehicle on the board, doc id = `String(entry.id)`.
  Fields: `id` (ms timestamp), `plate`, `vno` (property no.), `desc`, `who` (assigned person), `repair` (Preventive / Corrective / Express Maintenance), `status` (`bay`, `wait` or `sched`), `since` (ms timestamp, reset when moved between columns), `limitHrs`, `bay` (text such as "Bay 1"), `mechanics` (array, max 5 names).
  `work` (work description text, shown on the customer board), `dateIn` ('YYYY-MM-DD', date the vehicle entered the bay area; set on add, defaults to today, editable in Staff panel, never reset by dragging). `remarks` (free text such as "Waiting for parts", shown on the customer board under Work, editable in Staff panel). `schedDate` ('YYYY-MM-DD') and `schedTime` ('HH:MM', optional) for `sched` entries; a scheduled entry has `dateIn` '' until it is moved to Waiting/In bay (drag or Edit), which sets `dateIn` to today and resets `since`. Entries without `dateIn` fall back to the date of `id` (or `since` for small demo ids).
- End-user divisions: `meta/settings.divisions` = { normalized end-user name: division }, edited in Staff → Settings → "End-users and their divisions" (`renderDivisions()`). Entries may carry `division` only when it differs from the end-user's usual one (typed in the form's Division box); `entryDivision(e)` = `e.division || divisionOf(e.who)`. The form's End-user/Division boxes fill in when a vehicle is picked (`fillWhoFromVehicle`). Finished records store the division at the time of Done. Shown via `endUserHtml(e)` (`.enduser`) on cards and rows, in the Finished table, the print slip and the CSV.
- Doc `meta/roster` with `{ names: [...] }`, the mechanics list used by the pickers.
- Doc `meta/settings`: `pinHash` (SHA-256 of `'bay-status-board|' + PIN`; missing = default PIN 1234), `bays` (array, default Bay 1–4), `limits` ({repair type: hours}, defaults Preventive 2, Corrective 4, Express 1). Saved with merge; edited in the Staff panel.
- Entry hold fields: `hold` (reason text, '' = working), `holdAt` (ms the current hold started), `heldMs` (ms already on hold). Worked time = `workedMs(e)` = (holdAt or now) − since − heldMs; the limit, bar, near/over and the finished `bayMs` use worked time. History records also get `holdMs` (total time on hold). Moving lists resets `heldMs`.
- Doc `meta/clock` `{ t: serverTimestamp }`, written/read by each screen to measure its clock offset.
- Collection `history`, one doc per finished job, doc id = `String(entry.id)`: all entry fields plus `doneAt` (ms), `bayMs` (ms in bay = doneAt - since), `overLimit` (bool). Written by the Staff panel "Done" button (in-bay vehicles only) in one batch with deleting the `entries` doc. Read only while the Finished page is open (`subscribeHistory`, ordered by `doneAt` desc). Allowed by the existing `{document=**}` rule, so no rules change was needed.
- `VEHICLES` (127 items: `no`, `plate`, `desc`, `who`) is embedded in `index.html`. Vehicles added in the Staff panel are stored in `meta/settings.vehicles` (same fields); `rebuildVehicles()` merges them into `ALL_VEHICLES` (an added vehicle with the same property no. replaces the built-in details) and refills the search list. Use `ALL_VEHICLES`/`findVehicle()`, not `VEHICLES`, for lookups. Typos fixed 2026-11-05: Komatsu, Atlas Copco, Allan N. Hufana, Alvin-Barrey L. Taguicana, Julian M. Narvas, Jr. It powers the searchable vehicle field. Plate `N/A` means no plate, so the board shows the property number as the main label. The end-user (`who`) and their division are shown on the customer board (user asked for it 2026-11-06; TV option `who` can hide them).

## Layout (redesigned 2026-11-06)
- Design tokens at the top of the CSS (colors for Day/Night, `--r*` radii, `--shadow*`, `--gutter`). Bebas Neue only for display text (plates, clock, section titles); Inter for everything else, tabular numbers. Status colors: amber = working, red = over limit, blue = on hold/paused, green = live/done. Icons are inline SVG (`ICON`, `svgI()`), not emoji/unicode symbols, so they render on every device.
- Header: logo mark + title, count chips (`#counts`, `.stat`), clock + date (`#today`) + theme button + connection, and the action buttons Finished / TV mode / Staff in `.nav` (they keep the `staff-toggle` class, which TV mode and the Finished page hide). There are no floating buttons any more.
- Section titles carry a count (`#bayCount`, `#queueCount`, `#schedCount`) and the TV page indicator.
- Bay card: plate + bay tag; make & model · property no. (one line); chips (short repair type + status: In progress / N min left / Over limit / hold label); work/remarks; mechanic chips with a person icon; footer (`.card-foot`) with big elapsed time, "of Nh limit", progress bar, "In <date>" and time left / over by / timer paused. Cards must not get `overflow:hidden` (it lets them shrink and clip in the TV grid).
- Waiting/Scheduled rows: `.q-main` (plate + hold tag, make & model, work) and `.q-side` (repair chip, date in or scheduled day). Left border color shows hold/today/overdue.
- Staff panel: a wide dialog with a sticky header, tabs "Vehicles" (form card + the three lists) and "Settings" (`setStaffTab()`; roster, bays, default limits, vehicle list, TV display, security). Edit switches to Vehicles. Esc or ✕ (`#fClose`) closes it. Every form field has a label.
- Mechanics are picked with chips (`#mechPick`) that toggle the hidden `#fMechanic` multi-select; call `renderMechChips()` after changing its selection in code.

## Behaviors to preserve
- Board layout: In the bay now (left); Waiting, then Scheduled (right column). Waiting rows show only Date in (the waiting timer was removed at the user's request). Scheduled rows are sorted by date/time and labelled Today (amber), Tomorrow, weekday date, or Overdue (red).
- Staff panel has three columns: Scheduled, Waiting, In bay. Each row has an amber "Move to…" dropdown (the other two lists: In bay / Waiting / Scheduled) because dragging was hard and does not work on touch screens; the user asked for a dropdown rather than separate buttons. Choosing Scheduled asks for the date. Dragging still works on computers. Both go through `moveTo()`.
- Vehicles can also be dragged directly on the main board (not in TV mode, mouse only): drop zones `#bayZone`, `#queueZone`, `#schedZone`. While dragging, `boardDragging` pauses the 1-second `renderBoard()` redraw so the dragged card isn't replaced. Moving into Scheduled (board or Staff drag) asks for the date with a prompt (default tomorrow); cancelling leaves the vehicle where it was.
- On hold: Staff row "Put on hold…" dropdown (Awaiting parts / Under observation / At outside shop / Other reason…, or Resume work). Bay cards on hold are blue with the reason tag and "Elapsed (paused)"; waiting rows show the reason tag.
- Pause (can't finish today): In-bay Staff rows have "⏸ Pause" / "▶ Resume". Pause asks which day work continues (default `nextWorkday()`, skipping Sat/Sun) and sets `hold = 'Paused'` (`PAUSED`) plus `pauseUntil` ('YYYY-MM-DD'); it uses the hold timer, so paused time doesn't count. The board tag uses `holdLabel(e)`: "Paused · continues tomorrow / today / Mon, Nov 9". Resume = `setHold(e, '')`. Any other hold reason clears `pauseUntil`.
- Bay is a dropdown from `settings.bays` (old free-text bays are kept as an extra option when editing). Adding/editing into an occupied bay asks for confirmation (`bayFreeOk`).
- Time limit fills in from the repair type's default unless staff typed one (`limitTouched`).
- Work and remarks are cut to 2 lines on the board (`.wtext`); full text in Staff panel, tooltip, and Finished page.
- Elapsed time ticks every second; the card turns red with "OVER LIMIT" past the limit.
- Staff panel has two columns (Waiting, In bay); vehicles are dragged between them. Dragging resets the timer (`since`).
- Every field is editable at any time, including while waiting, via the **Edit** button on each vehicle in the Staff panel (In bay and Waiting). Edit loads the vehicle into the top form (title "Edit …", "Save changes" / "Cancel edit"); it can change the vehicle (duplicate guard applies), repair type, work, remarks, date in, status, bay, mechanics, time limit and the timer start (`fSince`). Changing status without touching the timer start resets `since`, like dragging. The Staff rows show a read-only summary; the old inline editors were replaced.
- Adding a vehicle already on the board (same property no.) is blocked.
- All user-supplied text goes through `esc()` before `innerHTML`, because the data is public and shared.
- The Staff panel is not rebuilt while someone is typing in it (`staffBusy()`), so live updates don't wipe input.
- Shared clock: every screen measures its offset from Firebase's server time (`serverOffset()`, every 30 min) and uses `nowMs()` instead of `Date.now()` for `since`, `doneAt`, timers and the header clock. Use `nowMs()` for any new time logic. `id` still uses `Date.now()` (just a unique id).
- Staff PIN: the Staff button, board drag, Finished-page Undo/Delete and CSV export need the device to be unlocked (`requireUnlock()`, `isUnlocked()`). Unlock is remembered per device in localStorage (`bayBoardUnlock` = current pin hash), so changing the PIN locks every other device. "Lock this device" and "Change staff PIN" are in the Staff panel. The default PIN is not accepted until settings have loaded (`settingsLoaded`). Viewing the board and Finished page needs no PIN. Forgot PIN: delete the `pinHash` field of `meta/settings` in the Firebase console, which resets it to 1234.
- Connection status under the clock (Live / Connecting / Offline) from snapshot metadata (`includeMetadataChanges`); a red offline bar shows after 8 s offline, with the last update time.

## TV display mode
- Turned on by `?tv` in the URL (kept on refresh) or the "TV mode" button; `body.tv` class drives the bigger CSS. "Exit TV mode" (top right, hidden with the cursor after 3s idle) turns it off.
- First tap/key enables full screen, the Web Audio chime and a screen wake lock (browsers require a user gesture). The board still shows without the tap.
- Red alert bar lists over-limit vehicles. Chime plays once per vehicle when it newly crosses the limit, not for vehicles already over when the page loads (`alerted` set).
- "N MIN LEFT" amber badge within `NEAR_MS` (15 min) of the limit; this shows in normal mode too.
- No scrolling on the TV: `body.tv` is exactly the screen height and each list (`#bayGrid`, `#queueList`, `#schedList`) shows only what fits. `paginate()` groups items into rows and rows into pages after every `renderBoard()`; pages flip every 15 s (`tvPageTick`), with "Page 1 of 3" and dots in the section title. (The old auto-scroll was removed.)
- Day/Night colors: `applyTheme()` sets `data-theme` (`light`/`dark`) on `<html>`. Staff screens: "◐ Auto / ☀ Day / ☾ Night" button under the clock, per device in localStorage `bayBoardTheme` (default `auto`). TV: `meta/settings.tv.theme` chosen in Staff panel → TV display (default `dark`). `auto` = Day 6 AM–6 PM (`DAY_START`/`DAY_END`), re-checked every minute. Day colors use slightly darker amber/blue/red for contrast on white.
- Header counts (`#counts`, all modes): in the bay · waiting · on hold (if any) · scheduled today.
- TV display options in the Staff panel (`meta/settings.tv`, key=false hides it, TV only): mechanic names (`.mechanic-block`), property no. (`.pno`), make & model (`.vd`, waiting/scheduled `.vdesc`), work (`.w-work`), remarks (`.w-rem`), date in (`.datein-row`, `.qrow .datein`). Body classes `tvh-<key>`.

## Finished maintenance page
- `#finished` in the URL (or the "Finished" button) shows the page instead of the board; "Back to board" clears the hash. Hidden in TV mode.
- Filters: text search, finished from/to date, repair type. Summary line counts jobs, within/over limit, average time in bay.
- Per row: Print (opens a one-page job record with signature lines: Mechanic / Inspected by / Received by), Undo (puts it back in the bay with its original `since`, blocked if the vehicle is already on the board) and Delete.
- Soft delete: Delete sets `deletedAt` on the history doc instead of deleting it. "Recently deleted" (`showBin`) lists those with Restore; records deleted more than 30 days ago are removed for good when the Finished page loads (`purgeOldDeleted`). Staff-panel Remove also goes to the bin: the entry is written to `history` with `deletedAt` and `removedFromBoard: true` (restore puts it back on the board).
- Summary (collapsible, follows the filters): by month (jobs, PM/CM/Express, % within limit, average time) and by mechanic.
- Vehicle history: "History" button on each Staff row, and clicking a vehicle on the Finished page, filter the list to that vehicle's property no. (or plate).
- "Export to Excel (CSV)" downloads the filtered list (UTF-8 BOM so Excel reads it). The CSV includes the end-user; the on-screen table does not.
- "Remove" in the Staff panel is for mistakes (goes to Recently deleted, not the finished list); "Done" is the normal way to finish a job.
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
- 2026-11-05: move buttons replaced by a "Move to…" dropdown.
- 2026-11-05: user asked to do all four groups of improvements. Part 1 done: staff PIN, offline warning, shared server clock (tested in demo mode and against a mock Firestore with a skewed server clock and offline switch).
- 2026-11-05: Part 2 done: hold/awaiting parts with paused timer, bay picker + bay settings, default limits per repair type, 2-line clamp on the board.
- 2026-11-05: Part 3 done: monthly/mechanic summary, vehicle history, printable job record, Recently deleted (30 days) for finished records and removed vehicles.
- 2026-11-05: Part 4 done: vehicle-list typos fixed; add/replace/remove vehicles from the Staff panel. All four improvement groups are finished.
- 2026-11-05: TV improvements "number 2" done: pages instead of scrolling, always dark, header counts, TV display options. Other TV ideas offered and not yet chosen: ready-by time and Ready for release list, customer status words, announcement ticker, voice announcement, repeat chime/quiet hours, nightly auto-refresh, after-hours screen, QR code.
- 2026-11-05: added Day/Night/Auto colors (staff button + TV setting).
- 2026-11-05: added Pause/Resume for jobs that continue another day.
- 2026-11-06: professional layout redesign (tokens, header with actions, new cards/rows, Staff panel with tabs and labeled form, mechanic chips). Tested desktop/phone/TV, Day and Night; all earlier feature tests re-run.
- 2026-11-06: end-user + division on the board; divisions entered per end-user in Settings (the user's list had no divisions; they may send one to prefill).
- Ideas the user has not decided on: link to the user's PM/CM maintenance records.
