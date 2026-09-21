# Dispatch — Work Queue Console

Dispatch is a single-file web app for managing a personal work portfolio: log tasks with priority, difficulty, and time estimates, and let the app suggest what to work on next based on a scoring system — or just ask it in plain language.

> First and foremost, Dispatch is a focus aid to help re-center yourself when there are too many tasks demanding your attention or you are distracted and need to get back on track. It is designed to meet you where you are in the moment, not demand your best at all times. It is meant to be a starting point from which you can create your own flavor of solution to track and deliver your work the way that works best for your brain.

It's a single HTML file on purpose — fork it, gut what doesn't help, build on what does. There's no "correct" way to use Dispatch; if a rule or feature here fights how your brain actually works, change it.

**Current version: v3.1.0** (shown in the app header; see [Version History](#version-history) below)

Dispatch now has two modes, toggled by a pill switch in the header, both sharing one task list, one Cloud Sync bin, everything:

- **Dispatch mode** (default) — for when you don't know where to start. Quick Match, Ask the Queue, and auto-suggested Next Up all surface something for you.
- **Aperture mode** — for when you already know what you need to do and the structure to do it in. A Session Builder replaces Quick Match; you hand-pick and order specific tasks, then start a Focus Timer to run through them (a la the Pomodoro technique: https://en.wikipedia.org/wiki/Pomodoro_Technique).

## Features

- **Task tracking** — title, notes, priority, difficulty, estimated time, due date, tags.
- **Priority scale** — five named levels instead of raw numbers: **Top, High, Medium, Low, Whatevs** (Top = highest). Only one task can be Top at a time; setting a second Top prompts you to confirm the swap.
- **Difficulty gauge** — a half-moon dial (Easy → Hard) instead of a plain slider, color-coded on the same scale as priority.
- **Automatic scoring & "Next Up"** — every task gets a numeric score from priority, difficulty, time, and due-date urgency; the top-scoring task is surfaced as a fully editable card (title, notes, priority, difficulty, time estimate, recurrence, tags all editable in place with auto-save), with a "Reroll" option.
- **Quick Match** — four one-tap buttons (Low/High Energy × Little/Lots of Time, split at difficulty 3 and 30 minutes) that instantly set Next Up to the best-scoring task matching that combination — no API call, no cost. If the chosen combination has no matching task, it cascades down through the other combinations (checking Low Energy + Little Time last) before giving up.
- **Set as Next Up** — every active task's ticket in the main list has a button to manually make it the Next Up pick, overriding whatever Ask the Queue, Quick Match, or the default scoring would have shown — useful when you already know what you want to work on while browsing the list.
- **Status actions**:
  - **Done** — closes the task (can be manually reactivated later from the Closed section if needed).
  - **Done for Day** — removes it from today's queue; it reactivates automatically the next calendar day.
  - **Hold…** — set it aside for a quick duration (1 hr / 4 hr), a chosen number of Days or Weeks, or a specific calendar day; it reactivates automatically once the hold expires. You can also start a task on hold right from the New Task / Edit Task form (and from Quick Add) using the same options, instead of creating it active and holding it separately afterward.
  - **De-prioritize** — sets it aside indefinitely until manually reactivated.
- **Recurring tasks**:
  - **Daily** — reactivates the next day after being marked Done.
  - **Weekly** — pick one or more days of the week; reactivates on the next occurrence of any selected day after being marked Done.
- **Chat assistant ("Ask the Queue")** — describe how much time or energy you have, and it recommends the best-fitting active task, using the Anthropic API. Supports voice input (browser speech recognition) and optional spoken replies. Only your active (non-Hold/Parked/De-prioritized/Closed) tasks are ever sent to the API. In standalone mode with a personal API key, each message typically costs well under a cent (Haiku 4.5 pricing, compact task format) — cost scales mainly with how many active tasks you have, since the whole active list is resent each message.
- **Undo** — a header button reverts the single most recent change (status change, edit, hold, delete, or import overwrite) for catching misclicks. Single-level only, and automatic background changes (like a Daily task auto-reactivating) don't count toward it.
- **Cloud Sync** — optional real cross-device sync via a free [JSONBin.io](https://jsonbin.io) bin: Dispatch loads from the cloud on open and pushes on every save, so anything added elsewhere (like from Quick Add) shows up the next time you open or refresh Dispatch. Pushes do a **fetch-merge-push** — every task is stamped with an `updatedAt` timestamp, and syncing keeps whichever copy of each task is newer rather than blindly overwriting the whole bin, so a stale or long-idle session can't silently erase tasks added elsewhere. Pushes are also batched (3-second debounce), so the UI never waits on the network. A **Sync Now** button in the header lets you pull in changes on demand. Falls back to the local cached copy if the network is unavailable. **Known limitation:** deleting a task on one device can still be "resurrected" by a stale session's merge, since there's no tracking yet of intentional deletions — this is a rarer and less damaging edge case than the overwrite bug the merge logic fixes, but worth knowing about. **Note:** Cloud Sync only works in the standalone version of Dispatch (downloaded file or hosted copy) — Claude's own artifact sandbox blocks outbound requests to third-party APIs like JSONBin, the same restriction that blocks microphone access there. See [Cloud Sync Setup](#cloud-sync-setup) below.
- **Quick Add companion page** (`quick-add.html`) — a separate page for adding a task from your phone without opening the full app. Exposes the same fields you'd see adding a task in Dispatch — title, notes, priority, difficulty, minutes, due date/Daily/Weekly, tags, and Start on Hold — defaulting to Medium priority, Medium difficulty, and 30 minutes if you don't touch them. Writes directly to the same shared bin as Cloud Sync. Great as a home-screen bookmark.
- **Export / Import** — download your task list as a `.json` file (also copied to clipboard) and import it elsewhere. **Import fully replaces your current task list** with the imported file's contents — it does not merge — and shows a confirm-before-wipe review step (with counts) before anything is deleted.

## Aperture Mode

Toggle to **Aperture** in the header pill switch to swap Dispatch's passive-suggestion tools (Quick Match, Ask the Queue) for an active sequencing workflow. Everything else — the task list, Undo, Cloud Sync, Export/Import — stays exactly the same underneath; only the panel beside Next Up and Next Up's own behavior change.

- **Session Builder** (in the panel where Quick Match sits in Dispatch mode) — tap **"+ Add to Session"** on any active task's ticket (next to "Set as Next Up") to queue it, or **"Add All (highest score first)"** to queue your entire active list at once, sorted by score (this replaces whatever's currently built rather than appending to it). Drag items in the list to reorder them. A built-but-not-started session is remembered (in that browser's `localStorage`) until you start it or hit Clear — closing the tab won't lose it.
- **A task drops out of the queue automatically** the moment it's marked Done, put on Hold, De-prioritized, or set to Done for Day — it won't linger as a stale entry once it's no longer active.
- **Focus Timer, decoupled from task completion** — pick a work/break preset (25/5, 25/10, 20/10, 15/5, 50/10, or 45/15 minutes) to start a repeating Pomodoro cycle: work → break → work → break, with a long break (3× the short break) automatically substituted every 4th work block. **The timer runs as its own independent rhythm** — it never pauses, skips, or waits on you finishing a task. **Next Up, separately, advances the instant your current task leaves active status** — pulling in whatever's next from the Session Builder immediately, regardless of what phase the timer happens to be in. Block length comes from the preset, not each task's own time estimate, to keep the planning step light.
- While a session runs: Quick Match/Ask the Queue's old spot and the task list hide, leaving only Next Up (fully editable) and the countdown. On every timer transition (work ending, break ending): a two-tone bell repeated 3 times (pause between each half the bell's own length, ~7s total), synced with a matching pulse on the card.
- **The session ends the instant the Session Builder queue is empty** — not at the next block boundary — showing a "session complete" message and returning to normal view. **"End Timer"** is a subtle underlined link that ends the whole session manually at any point.
- **The mode toggle locks while a session is running** (both buttons are disabled — end or complete the session first) so you can't orphan a running timer by switching modes mid-session.

## Getting Started

### Option A: Run it inside Claude (recommended if you want to use the Ask the Queue chat feature)

Open `dispatch.html` as a Claude artifact (e.g. by uploading it into a claude.ai conversation, or continuing to use the version already shared with you there). This is the only mode where:
- Task and chat data **persist automatically** and can sync across devices logged into the same Claude account.
- The **chat assistant** works, since it calls the Anthropic API using Claude's built-in credentials — no API key setup needed.
- **Cloud Sync will not work here** — Claude's artifact sandbox blocks outbound requests to third-party APIs like JSONBin (the same restriction that blocks microphone access). Use the standalone version below if you want Cloud Sync.

### Option B: Run it standalone in any browser

Just open `dispatch.html` directly (double-click it, or drag it into a browser tab). In this mode:
- Task and chat data are saved to that browser's **`localStorage`** automatically — persisting across sessions on that device/browser, but **not syncing** to other devices or browsers (See Cloud Sync Setup below for steps on setting up shared storage).
- The **chat assistant will not work** — it calls `https://api.anthropic.com/v1/messages` directly from the page, which only succeeds inside Claude's own environment out of the box, **unless** you supply your own Anthropic API key. When running standalone, a small "Ask the Queue" settings box appears where you can paste a key from [console.anthropic.com](https://console.anthropic.com); it's stored only in that browser's `localStorage` (via the `anthropic-dangerous-direct-browser-access` header for a "bring your own key" client-side pattern) — it is never written into this file, uploaded, or included in an Export. **Never hardcode a real API key into `dispatch.html` itself, especially before committing it to a public repo** — always enter it through that box at runtime instead. Everything else (task tracking, scoring, hold/daily/weekly, export/import, voice input for dictation) works fully offline regardless.

### Moving data between copies

Use the **Export** button to download a `.json` snapshot of your tasks (or copy it from your clipboard), then **Import** it into another copy of the app — on another device, another browser, or after switching between the Claude and standalone versions. **Import replaces the entire current task list** with the file's contents; it does not merge, and anything not in the imported file is permanently deleted. A review step shows exactly how many tasks will be deleted and added before you confirm.

## Cloud Sync Setup

For real cross-device sync (not just manual export/import), Dispatch can sync through a free [JSONBin.io](https://jsonbin.io) bin. This is optional — without it, Dispatch just uses local storage as described above.

1. Create a free account at **jsonbin.io**.
2. Go to the **API Keys** page and create a new **Access Key** (not the Master Key) — name it something like "Dispatch", and grant it **Bins: Read + Update + Create** (leave Delete unchecked). Create is needed for the "Create New Bin" button in Dispatch; Delete is deliberately withheld so a leaked key can't destroy your data. JSONBin explicitly recommends against using the Master Key in frontend code, since it has full, unrestricted account access.
3. In Dispatch, click **☁ Cloud Sync** in the header, paste in your Access Key, and either click **Create New Bin** (Dispatch will create one and fill in the Bin ID for you) or paste in an existing Bin ID if you've already set one up. Click **Save & Sync**.
4. Repeat step 3 on any other device/browser running Dispatch, using the **same Bin ID** each time, to connect them to the same shared task list.

If "Save & Sync" fails, the error message now shows the exact reason (HTTP status and JSONBin's own message) rather than a generic failure — usually a missing permission on the Access Key, or a mismatched Bin ID.

Your Access Key and Bin ID are stored only in that browser's `localStorage` — never written into `dispatch.html` itself, so it's safe to keep the file in a public repo. Unlike the Anthropic key, a scoped JSONBin Access Key is generally fine to note down/reuse across devices since it can only read and update this one bin, not delete it or touch anything else on your account — but treat it as a credential worth keeping private regardless.

### Quick Add from your phone

`quick-add.html` is a second, separate file — a minimal page with just a text box and an "Add Task" button, for logging a task without opening the full Dispatch app. It writes directly to the same JSONBin bin as Cloud Sync.

To use it:
1. Host `quick-add.html` somewhere reachable from your phone. The easiest option is **GitHub Pages** — enable it on this repo (Settings → Pages), which gives you a public URL like `https://<username>.github.io/<repo>/quick-add.html`.
2. Open that URL on your phone and bookmark it to your home screen (in most mobile browsers: Share → Add to Home Screen) so it opens like a mini app with one tap.
3. The first time you open it, it'll ask for the same Access Key and Bin ID you used in Dispatch's Cloud Sync setup — enter them once and it remembers them on that device.
4. From then on: tap the icon, fill in your task (title required, everything else optional and defaulted), tap Add. It shows up in Dispatch the next time you open it, refresh it, or tap **Sync Now** in the header.

## How Scoring Works

```
score = (6 − priority) × 22 − difficulty × 6 − min(estMinutes, 240) / 240 × 20 + bonus
```

- **Priority** dominates: Top contributes +110, Whatevs only +22.
- **Difficulty** and **estimated time** apply small penalties, favoring easier/shorter tasks as a tiebreaker.
- **Bonus**: +45 if overdue, +32 if due today/tomorrow, +16 if due within 3 days, +6 if due within a week, or a flat **+33** if the task is marked Daily or Weekly (edging out same-priority tasks due today/tomorrow).

## Version History

| Version | Changes |
|---|---|
| v1.0.0 | Initial build: task tracking, scoring, hold/park states, chat assistant, voice input |
| v1.1.0 | "Top" priority exclusivity rule |
| v1.2.0 | Priority renamed to named levels; scoring direction reversed (1 = highest) |
| v1.3.0 | Vertical priority buttons + difficulty gauge, color-coded |
| v1.4.0 | Daily recurrence |
| v1.4.1 | Weekly recurrence (single day) |
| v1.4.2 | Weekly recurrence: multi-day select |
| v1.5.0 | Export/Import; environment-aware storage (Claude storage with localStorage fallback) |
| v1.6.0 | Local-timezone day boundaries; day-only Hold option; version display added |
| v1.6.1 | Added "Reactivate" button for closed (Done) tasks |
| v1.6.2 | Task add/edit modal no longer closes when clicking outside it |
| v1.6.3 | Voice input errors now surface a message instead of failing silently |
| v1.7.0 | Standalone mode can use the chat assistant via a user-supplied Anthropic API key (browser-local only) |
| v1.7.1 | Cost reduction: compact task format (~50% fewer tokens) and Haiku 4.5 for standalone mode (~3x cheaper) |
| v1.7.2 | Fixed "due today/tomorrow" and score bonus flipping around noon (calendar-date comparison instead of elapsed time) |
| v1.8.0 | Added Difficulty (easiest/hardest first) sort options; toggleable Daily/Weekly filter buttons |
| v1.9.0 | Added "Quick Match" quadrant buttons — instant, no-API alternative to Ask the Queue that sets Next Up based on energy/time |
| v2.0.0 | Next Up card is fully editable in place (title, notes, priority, difficulty, time, recurrence, tags) with auto-save |
| v2.1.0 | Quick Match moved beside Next Up (still a 2x2 grid); added escalating color coding across the four quadrant buttons |
| v2.1.1 | Quick Match buttons are now square; Next Up notes field defaults to a 6-line height |
| v2.1.2 | On Hold and Parked for Today sections now start collapsed by default |
| v2.2.0 | Import now fully replaces the task list instead of merging, with a confirm-before-wipe review step |
| v2.3.0 | Redesigned Hold options: 1hr/4hr quick buttons plus a Days/Weeks stepper (Days 1-6, Weeks 1-4) for longer holds |
| v2.4.0 | Added single-level "Undo" button in the header for misclicks |
| v2.5.0 | Added Cloud Sync (JSONBin.io) for real cross-device sync, plus quick-add.html companion page for phone quick-capture |
| v2.5.1 | Fixed Cloud Sync connection failures (correct auth header, defensive parsing, real error messages) |
| v2.5.2 | quick-add.html now uses the same favicon as Dispatch |
| v2.6.0 | Removed automatic 30-second Cloud Sync poll (was eating JSONBin quota); added manual "Sync Now" button |
| v2.7.0 | Cloud Sync push is now non-blocking and batched (3s debounce) instead of blocking the UI on every action |
| v2.8.0 | Added "Set as Next Up" button to task cards; quick-add.html now exposes all task fields instead of title-only |
| v2.8.1 | "Set as Next Up" moved to its own bottom-left row, styled amber like "+ New Task" |
| v2.9.0 | Fixed data-loss bug: Cloud Sync now does fetch-merge-push instead of blindly overwriting the bin, so a stale session can't erase tasks added elsewhere |
| v3.0.0 | Added "Start this task on hold" to task creation/editing and to quick-add.html, matching the standalone Hold options |
| v3.1.0 | Merged Aperture back in as a mode (Session Builder + decoupled Focus Timer) instead of a separate forked app, toggled by a header pill switch |

## Notes & Limitations

- Dispatch itself (`dispatch.html`) is still a single self-contained HTML file — no build step, no dependencies to install. `quick-add.html` is a separate, optional companion file (see Cloud Sync Setup above) — you only need it if you want phone quick-capture.
- **Security note**: if you use the standalone BYO-API-key option, that key lives only in your browser's `localStorage` — it is never part of this file's source. Don't paste a real key anywhere in `dispatch.html` itself before committing it to version control.
- Voice input relies on the browser's built-in speech recognition (best support in Chrome/Edge; limited in Firefox/Safari). When run inside Claude's artifact preview, the sandboxed environment may block microphone access entirely — if voice input doesn't work there, try the standalone downloaded file in a regular browser tab instead.
- There's no encryption or account system — anyone with access to the file/browser profile can see the stored tasks.
