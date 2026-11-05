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
- If there are more vehicles than fit on the screen, each list shows **pages** ("Page 1 of 3") and flips to the next page every 15 seconds. Nothing scrolls.
- **TV colors:** Night (dark, the default), Day (light), or Auto (Day from 6 AM to 6 PM, Night after). Choose in the Staff panel under **TV display → TV colors**. The TV changes within a few seconds.
- The top of the board shows the counts: in the bay, waiting, on hold, and scheduled today.
- **Choose what the TV shows:** in the Staff panel under **TV display**, untick what you don't want on the lobby TV (mechanic names, property no., make & model, work, remarks, date in). This only changes the TV; staff screens still show everything.

To leave TV mode, move the mouse and click **Exit TV mode** (top right).

## Moving vehicles
In the Staff panel, each vehicle has an amber **Move to…** dropdown. Pick **In bay**, **Waiting** or **Scheduled** and it moves right away (Scheduled asks for the date). Moving restarts the vehicle's timer. On a computer you can also drag a vehicle with the mouse, either in the Staff panel or directly on the main board, between **In the bay now**, **Waiting** and **Scheduled**. Dropping into Scheduled asks for the date. (Dragging doesn't work on phones or tablets, and is turned off in TV mode.)

## Scheduling a vehicle
1. Click **Staff**, choose the vehicle, and set the status box to **Scheduled (future date)**.
2. Pick the **Scheduled date** (and a time if you have one), then click **Add to board**.
3. It shows on the board under **Scheduled**: "Today" in amber, "Tomorrow", the date, or "Overdue" in red if the date has passed.
4. When the vehicle arrives, open its **Move to…** dropdown in the Staff panel and pick **Waiting** or **In bay**. (Dragging also works on a computer.) Its date in becomes today and its timer starts.

## Awaiting parts (on hold)
1. In the Staff panel, open the blue **Put on hold…** dropdown on the vehicle and pick **Awaiting parts**, **Under observation**, **At outside shop**, or **Other reason…**.
2. On the board the card turns blue with the reason, and the timer **pauses**, so time on hold doesn't count against the time limit.
3. When work starts again, open the same dropdown and pick **Resume work**.
4. On the Finished page, the time in bay is the working time, with the time on hold shown underneath.

## Bays and default time limits
- **Bay** is now a dropdown. Add or remove bays in the Staff panel under **Bays**. If you choose a bay that already has a vehicle, the board asks you to confirm.
- Each repair type has a **default time limit** (Preventive 2h, Corrective 4h, Express 1h to start). It fills in automatically when you pick the repair type. Change the defaults in the Staff panel under **Default time limit**. You can still type a different limit for one vehicle.

## Fixing a mistake (Edit)
1. Click **Staff** and find the vehicle in the **Waiting** or **In bay** list.
2. Click the blue **Edit** button. The vehicle's details load into the form at the top.
3. Change what is wrong. You can change the vehicle, repair type, work, remarks, date in, In bay/Waiting, bay, mechanics, time limit, or, if the timer is wrong, the time it started.
4. Click **Save changes**, or **Cancel edit** to leave it as it was.

## Finishing a job and the Finished list
1. When a job is finished (even early), click **Staff**, find the vehicle in the **In bay** list, and click the green **Done** button. Click **OK** to confirm.
2. The vehicle leaves the board and is saved in **Finished maintenance**, with the time it spent in the bay and whether it was within the time limit.
3. To see the list, click **Finished** at the bottom right of the board. You can search, pick a date range or repair type, and click **Export to Excel (CSV)** to download it.
4. Pressed Done by mistake? On the Finished page, click **Undo** on that row to put the vehicle back in the bay.
5. **Remove** in the Staff panel is only for vehicles added by mistake. It does not add anything to the Finished list, but the vehicle is kept in **Recently deleted** for 30 days.
6. **Summary:** on the Finished page, click **Summary by month and by mechanic** to see jobs per month (PM / CM / Express), the share finished within the limit, and the average time, plus the same per mechanic. It follows the search and date filters, so you can pick a date range for a report.
7. **Vehicle history:** click **History** on a vehicle in the Staff panel, or click a vehicle's name on the Finished page, to see all its past jobs.
8. **Print:** click **Print** on a row to print a one-page job record with signature lines (Mechanic, Inspected by, Received by). If nothing opens, allow pop-ups for the site.
9. **Delete and Recently deleted:** Delete moves a record to **Recently deleted** (button at the top of the Finished page). It stays there for 30 days, with a **Restore** button, and is then removed for good.

## Adding a vehicle to the list
If a vehicle isn't found when you search:
1. Click **Staff** and scroll down to **Vehicle list**.
2. Type the **Property no.**, the **Plate no.** (or N/A), the **Make & model**, and the **End-user** if you know it.
3. Click **Add to vehicle list**. It can now be searched on every screen.
- Typing a property no. that is already in the list replaces its details, for example a new end-user. Click **Remove** to go back to the original details.

## Day and Night colors
On staff computers and phones, click the small button under the clock to switch between **☀ Day** (light), **☾ Night** (dark) and **◐ Auto** (Day from 6 AM to 6 PM, Night after). Each device remembers its own choice.

## Staff PIN
- The **Staff** button, dragging on the board, and Undo/Delete/Export on the Finished page need the staff PIN. Looking at the board and the Finished list does not.
- The starting PIN is **1234**. Change it right away: click **Staff**, enter 1234, scroll down to **Security**, and click **Change staff PIN**.
- Each device asks for the PIN once and then remembers it. Click **Lock this device** (Staff panel → Security) on a shared or public computer. When the PIN is changed, every other device asks for the new one.
- Forgot the PIN?
  1. Open the Firebase console and go to **Firestore Database** → **Data**.
  2. Open the **meta** collection, then the **settings** document.
  3. Delete the **pinHash** field. The PIN goes back to 1234.

## Connection and clock
- Under the clock, the board shows **Live** when it is getting updates. If the internet drops, it shows **Offline** and a red bar with the time of the last update.
- Timers use Firebase's clock, so every screen shows the same times even if one device's clock is wrong.

## Notes
- Until `apiKey` is filled in, the page runs in demo mode (data stays on that one device) and shows a banner saying so.
- The Staff button only hides the controls; with the open rules above, anyone who has the link could edit. A simple PIN on the Staff panel can be added if you want it.
