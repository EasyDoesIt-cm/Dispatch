# Dispatch — Work Queue Console

Dispatch is a single-file web app for managing a personal work portfolio: log tasks with priority, difficulty, and time estimates, and let the app suggest what to work on next based on a scoring system — or just ask it in plain language.

**Current version: v2.5.1** (shown in the app header; see [Version History](#version-history) below)

## Features

- **Task tracking** — title, notes, priority, difficulty, estimated time, due date, tags.
- **Priority scale** — five named levels instead of raw numbers: **Top, High, Medium, Low, Whatevs** (Top = highest). Only one task can be Top at a time; setting a second Top prompts you to confirm the swap.
- **Difficulty gauge** — a half-moon dial (Easy → Hard) instead of a plain slider, color-coded on the same scale as priority.
- **Automatic scoring & "Next Up"** — every task gets a numeric score from priority, difficulty, time, and due-date urgency; the top-scoring task is surfaced as a fully editable card (title, notes, priority, difficulty, time estimate, recurrence, tags all editable in place with auto-save), with a "Reroll" option.
- **Quick Match** — four one-tap buttons (Low/High Energy × Little/Lots of Time, split at difficulty 3 and 30 minutes) that instantly set Next Up to the best-scoring task matching that combination — no API call, no cost. If the chosen combination has no matching task, it cascades down through the other combinations (checking Low Energy + Little Time last) before giving up.
- **Status actions**:
  - **Done** — closes the task (can be manually reactivated later from the Closed section if needed).
  - **Done for Day** — removes it from today's queue; it reactivates automatically the next calendar day.
  - **Hold…** — set it aside for a quick duration (1 hr / 4 hr), a chosen number of Days or Weeks, or a specific calendar day; it reactivates automatically once the hold expires.
  - **De-prioritize** — sets it aside indefinitely until manually reactivated.
- **Recurring tasks**:
  - **Daily** — reactivates the next day after being marked Done.
  - **Weekly** — pick one or more days of the week; reactivates on the next occurrence of any selected day after being marked Done.
- **Chat assistant ("Ask the Queue")** — describe how much time or energy you have, and it recommends the best-fitting active task, using the Anthropic API. Supports voice input (browser speech recognition) and optional spoken replies. Only your active (non-Hold/Parked/De-prioritized/Closed) tasks are ever sent to the API. In standalone mode with a personal API key, each message typically costs well under a cent (Haiku 4.5 pricing, compact task format) — cost scales mainly with how many active tasks you have, since the whole active list is resent each message.
- **Undo** — a header button reverts the single most recent change (status change, edit, hold, delete, or import overwrite) for catching misclicks. Single-level only, and automatic background changes (like a Daily task auto-reactivating) don't count toward it.
- **Cloud Sync** — optional real cross-device sync via a free [JSONBin.io](https://jsonbin.io) bin: Dispatch loads from the cloud on open, pushes on every save, and polls every 30 seconds for changes made elsewhere. Falls back to the local cached copy if the network is unavailable. See [Cloud Sync Setup](#cloud-sync-setup) below.
- **Quick Add companion page** (`quick-add.html`) — a separate, minimal page (just a text box and a button) for adding a task from your phone without opening the full app. Writes directly to the same shared bin as Cloud Sync. Great as a home-screen bookmark. New tasks land with Medium priority, Medium difficulty, and a 30-minute estimate — easy to fine-tune later in Dispatch.
- **Export / Import** — download your task list as a `.json` file (also copied to clipboard) and import it elsewhere. **Import fully replaces your current task list** with the imported file's contents — it does not merge — and shows a confirm-before-wipe review step (with counts) before anything is deleted.

## Getting Started

### Option A: Run it inside Claude (recommended)

Open `dispatch.html` as a Claude artifact (e.g. by uploading it into a claude.ai conversation, or continuing to use the version already shared with you there). This is the only mode where:
- Task and chat data **persist automatically** and can sync across devices logged into the same Claude account.
- The **chat assistant** works, since it calls the Anthropic API using Claude's built-in credentials — no API key setup needed.

### Option B: Run it standalone in any browser

Just open `dispatch.html` directly (double-click it, or drag it into a browser tab). In this mode:
- Task and chat data are saved to that browser's **`localStorage`** automatically — persisting across sessions on that device/browser, but **not syncing** to other devices or browsers.
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
4. From then on: tap the icon, type a task, tap Add. It shows up in Dispatch (via Cloud Sync's ~30-second poll, or immediately the next time Dispatch is opened) with Medium priority, Medium difficulty, and a 30-minute estimate — open Dispatch to adjust any of that.

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

## Notes & Limitations

- Dispatch itself (`dispatch.html`) is still a single self-contained HTML file — no build step, no dependencies to install. `quick-add.html` is a separate, optional companion file (see Cloud Sync Setup above) — you only need it if you want phone quick-capture.
- **Security note**: if you use the standalone BYO-API-key option, that key lives only in your browser's `localStorage` — it is never part of this file's source. Don't paste a real key anywhere in `dispatch.html` itself before committing it to version control.
- Voice input relies on the browser's built-in speech recognition (best support in Chrome/Edge; limited in Firefox/Safari). When run inside Claude's artifact preview, the sandboxed environment may block microphone access entirely — if voice input doesn't work there, try the standalone downloaded file in a regular browser tab instead.
- There's no encryption or account system — anyone with access to the file/browser profile can see the stored tasks.
