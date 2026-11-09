# Flowdo — backlog

Features we want but haven't scheduled, grouped into batches that each ship as one release.
`ROADMAP.md` lists them in short; this file breaks each batch into the tasks that build it.
Nothing here is started. The roadmap stays paused after v0.5 until Navid restarts it, and a
batch moves into `ROADMAP.md` under "Next" when it's picked.

Added 2026-11-09 from a gap review: what a time tracker people rely on has that Flowdo doesn't.

Every batch meets the roadmap's release standard:
- typecheck, unit tests, a real build and the end-to-end suite on Windows and Linux;
- the sweep and `npm audit` gates;
- release notes in `docs/releases/`.

---

## Batch 1 — your hours, trustworthy

**Why first:** stats are only as good as the sessions under them. Today one forgotten Stop
leaves a six-hour "focus session" in your stats for good, and time away from the desk counts
as focus. v1.0's data-integrity work assumes clean data going in, so this batch belongs
before v1.0.

**1.1 Add and edit sessions** (deleting already works from Stats → history)
- [ ] `sessionsRepo.update(id, patch)` and a validated `create` for manual entries: start
      before end, not in the future, no overlap with another session, and a project or task
      that still exists.
- [ ] Mark manual sessions as manual (a migration adding `source = 'manual'`) so stats can
      show them honestly.
- [ ] IPC `sessions.update` / `sessions.addManual`, with input validated in main, not just in
      the window.
- [ ] Add/edit dialog, opened from the session history and by clicking a session block in
      the Day/Week timeline. Fields: date, start, end, project, task, kind, notes.
- [ ] "Add time" on the timeline, by dragging over an empty slot or using a button.
- [ ] Editing a running session is refused; the timer owns it until it ends.
- [ ] Tests: unit tests for the overlap and validation rules (including the DST edge), an
      e2e test that adds, edits and deletes a session and checks Stats totals move with it,
      and export/import round-trips manual sessions.

**1.2 Idle detection**
- [ ] Poll `powerMonitor.getSystemIdleTime()` while a focus phase runs (`src/main/power.ts`).
      Past a threshold (Settings: off / 5 / 10 / 15 min, default 10), remember when the idle
      time began.
- [ ] On return, ask: "You were away from 14:10 to 14:52. Keep it, discard it, or stop the
      session at 14:10?" Show this in the main window, and as a notification if the window
      is hidden.
- [ ] The timer applies the answer through the same wall-clock path that suspend/resume
      already uses, so nothing bypasses the session log.
- [ ] Wayland: check whether idle time is reported. If not, hide the setting with an
      explanation, as the global hotkeys already do.
- [ ] Tests: timer unit tests with an injected idle clock for keep, discard and stop-at, plus
      an e2e test driving a fake idle source.

---

## Batch 2 — goals and estimates

**Why:** Stats show what happened, but you can't see whether it was enough. Goals and
estimate tracking turn the numbers into something you act on daily.

**2.1 Daily and weekly goals**
- [ ] Settings: a daily focus goal (hours, or Pomodoros) and a weekly goal, both optional.
      The week follows `weekStartsOn`.
- [ ] A progress ring in the timer panel ("2h 40m of 4h today") and a goal line on
      the Stats daily chart.
- [ ] A quiet "goal reached" chime and notification, once per day, and only when enabled.
- [ ] Tests: unit tests for the progress maths (week boundaries, days with no sessions) and
      an e2e test that sets a goal and logs a session.

**2.2 Estimate vs actual**
- [ ] Show "3 of 5 Pomodoros" on tasks that have `estimatedPomodoros`, counting completed focus
      sessions on that task.
- [ ] Flag a task once it goes over its estimate.
- [ ] Stats: estimate accuracy over the chosen range (finished tasks, estimated vs used).
- [ ] Tests: a unit test for the count, which includes Flowmodoro sessions as their minute
      equivalent of Pomodoros, plus an e2e test.

---

## Batch 3 — reports you can hand to someone

**Why:** the only export today is a full backup. A per-project/per-task report for a date
range is what clients and managers ask for. It's also the local first step towards the
backlogged shared time tracking: a report needs no server or Clockify sync.

**3.1 Report export**
- [ ] A report builder in Stats: date range, then group by project / task / day, optionally
      filtered to one project.
- [ ] CSV export (one row per session or per group), with times in local time and the time
      zone stated in the header.
- [ ] PDF export via Electron's `printToPDF` of a print-styled report view: totals per
      project and task, and an optional session list.
- [ ] Save through the native save dialog. Nothing is uploaded.
- [ ] Tests: unit tests for grouping and totals (matching the Stats totals) and CSV escaping,
      plus an e2e test that exports both formats and parses the CSV back.

---

## Batch 4 — focus protection

**Why:** a focus app should keep the rest of the desktop quiet during a focus block. It
depends on the operating system, so it comes last.

**4.1 Do Not Disturb during focus**
- [ ] Research spike first: Windows Focus Assist / Do Not Disturb (whether a supported API
      exists for an unpackaged desktop app), and GNOME/KDE notification-silencing over D-Bus.
- [ ] If supported: a setting, "Silence notifications during focus", that turns on at focus
      start and is always restored at the end, on Stop and on quit, even if the app crashes
      (restore on next launch).
- [ ] If not supported on a platform, the setting says so instead of silently doing nothing.
- [ ] Tests: unit tests for the restore-on-crash bookkeeping. Real end-to-end checks happen
      on the hardware that supports it.

---

## How a batch runs

The same flow as v0.5.x:
1. Planned in a PR description.
2. Built by Sonnet agents in parallel on files that don't overlap; Haiku handles the version
   bump, docs and CI-log summaries.
3. Reviewed and gated by the orchestrator.
4. Released by merge on Navid's go-ahead.
