# Campus Copilot

Berkeley x Stanford Grok Bot Hackathon. Track: Best Student Use Case.

Someone taking several courses using different platforms (Canvas, Gradescope and so on) while also recruiting. Each week they otherwise dig through notification email for due dates, rebuild a study plan from what they missed, and scan job boards. Campus Copilot splits that into five Grok Bots.

| Bot | Role |
|---|---|
| Copilot | Manager. Talks to the student, messages the workers, merges `campus-brief.md`, and asks before any calendar export. |
| Deadline Scout | Reads all the notifications from different platforms (Canvas, Gradescope ..) and mails. Extracts the assignment, course, due time, and link. Builds an `.ics` only for rows Copilot says were approved. |
| Study Coach | Runs attempt-first review. One full question, the student tries, then a specific gap and a new question. Reports topic state. |
| Practice Builder | Overnight questions, a practice exam, and a cheat sheet sized to the professor's page rules. |
| Career Scout | Weekly jobs ranked against the resume, plus career fairs and events with dates and sign-up links. |

The dashboard in [dashboard/campus-copilot.html](mac-app/dashboard/) is the screen. The bots are the work. Copilot updates only the `seed()` data in that file. It does not restyle the page, and the chat box inside the page is a local demo script, not the live manager.

## How we built it

Each bot is a separate Grok Bot. Copilot was given its job in chat and told to remember it. Two skills were taught on a fixture, not on live mail:

- `approval-gate`. A fake Deadline Scout table (DATA 8 Homework 5, CS 189 Project 1, CS 189 midterm in 9 days) was shown and no `.ics` was built. DATA 8 was approved and the CS 189 project was rejected. The midterm stayed on the exam list. A rejected deadline is left out of the calendar and out of nothing else.
- `weekly-brief`. Four stale worker blocks were merged into one message: deadlines, the missed topic, the practice file, and one job plus one fair. Missing workers are named. Stale sections stay labeled stale.

Routines on Copilot, Pacific Time :

- Sunday about 6:00pm: weekly brief.
- Morning about 8:00am, including weekends: exam-week check-in, sent only when an exam is within 7 days.

Worker schedules, owned by those bots, not by Copilot:

- Deadline Scout, daily 7:00am.
- Study Coach and Practice Builder, nightly 1:00am, and again 3 days before an exam.
- Career Scout, Monday 8:00am.

Paste-ready jobs are in [bots/](bots/). The message shapes Copilot can merge are in [docs/handoff-contract.md](docs/handoff-contract.md).


The page cannot call a bot by itself. A webhook can wake Copilot, but the answer returns in the Grok Bot chat, not inside the page.
The approval list is easy to over-read. Rejecting a project deadline must not drop the exam, and an exam 9 days away must not start the morning check-in. The manager should refuse to invent a due date or a topic when a worker is missing. Partial approval has to be a sentence the student types (`approve DATA 8, reject CS 189`).

## How to run it

1. In the Grok Bot app, create Copilot and paste [bots/copilot.md](bots/copilot.md) as its first message. Send `Remember this as your job.`
2. Create the four workers, or import each teammate's template with **Add to Grok Bot**. Importing adds a copy. It does not replace Copilot. Paste the matching file in [bots/](bots/) if that worker does not already have its job.
3. Press `Ctrl+Shift+N` or `Cmd+Shift+N`, choose **New group chat**, and add all five bots in **To:**. Name it Campus Copilot. Tell them to reply in the group.
4. Drop [dashboard/campus-copilot.html](dashboard/campus-copilot.html) into Copilot's own chat and use the message in [docs/integration.md](docs/integration.md).
5. Ask `What's due this week?` Copilot must show Deadline Scout's table and wait. Approve one row and reject one. The `.ics` may contain only the approved row.
6. Open the HTML file Copilot sends back. Deadline, topic, job, and fair data should match `campus-brief.md`. Sample rows stay marked stale until a worker replies for real.

## Campus Copilot Mac App

Extension of the project to a Mac application [mac-app/](mac-app/)

A local macOS Electron wrapper for the Campus Copilot HTML dashboard. It keeps the exact original dashboard layout, CSS, and mascot, adds direct Google Drive folder sync (removing the manual "Send to team" copy step), and wires the in-app mascot chat drawer to two-way JSON files so Grok Bot can read and reply automatically. On launch (or via **Preferences**, `Cmd-,`), the app picks a sync folder — defaulting to `Campus Copilot` inside Google Drive for desktop (`~/Library/CloudStorage/GoogleDrive-*/My Drive/Campus Copilot`).

Files in the sync folder:

- `campus-copilot-state.json` — written by the app on every dashboard state change.
- `campus-copilot-changes.txt` — plain-text summary of changes since the last dashboard update, written on every change.
- `campus-copilot-history.log` — timestamped append-only log of dashboard changes.
- `dashboard.html` — optional live dashboard dropped in by Campus Copilot; the app watches it and reloads automatically.
- `campus-copilot-chat-outbox.json` — written by the app when the student sends a message from the mascot chat drawer.
- `campus-copilot-chat-inbox.json` — written by Campus Copilot and watched by the app (`fs.watch` + 2-second poll); replies appear in the drawer with sender tags and an unread indicator on the mascot when closed.

## Run locally

```bash
cd mac-app
npm install
npm start
```

## Build macOS `.zip` packages

```bash
cd mac-app
npm install
npm run build:mac        # builds both arm64 (Apple silicon) and x64 (Intel) into dist/
# or build one architecture:
npm run build:mac:arm64
npm run build:mac:x64
```

Builds are unsigned (`"identity": null`). On first launch, right-click `Campus Copilot.app` and choose **Open** (or clear quarantine if macOS warns the app is damaged):

```bash
xattr -cr "/Applications/Campus Copilot.app"
```

## Privacy note

`dashboard/dashboard.html` bundled here contains only the generic sample dataset (no personal student emails, course links, or grades). To use live student data, have Campus Copilot write `dashboard.html` into the synced Google Drive `Campus Copilot` folder instead of committing it to this repository.
