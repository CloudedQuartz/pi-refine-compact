# Changelog

Notable changes per version (Keep a Changelog format, SemVer).


## [0.2.1] - 2026-10-04

### Added

- **Session model as the summarizer.** `/compact-model` now offers three modes:
  `off — pi's native compaction` (the default, `model: null`), `session — follow
  the chat's model` (refines with whatever model the chat is using at compaction
  time, captured once per run), and any fixed `provider/id`. A session with no
  model falls back to native compaction with one info line.
- **Virtual-model guard** (777972f): virtual (router-only) catalog entries are
  filtered out of the `/compact-model` menu, and a summarizer that resolves to
  one — saved ref or session model — loudly falls back to stock compaction
  instead of failing the whole run.
- **Test suite** (`bun test`) covering session mode, the guard, backwards
  compatibility, and cache isolation.

### Fixed

- **Cached chunks no longer leak across sessions** (876613f): cache keys are
  scoped by session id, resolved summarizer and derived budgets, so switching
  any of them recomputes instead of reusing stale text. Guard idea from PR #1
  by CloudedQuartz.
- **Dead duplicate `throw` after the retry loop in `callSummarizer`** (1316fc2):
  a leftover from c018351 was misindented past the loop body.

## [0.2.0] - 2026-10-01

### Fixed

- **"length cap hit" — compaction no longer dies here.** The running summary
  could outgrow the reply budget, so compaction was guaranteed to fail no
  matter how chunks were split. The summary ceiling now always fits inside the
  reply budget.
- **No more phantom splitting on small sessions.** Prompt sizes were
  underestimated whenever tool calls carried commands, code, or file contents,
  so chunks got split up to three levels deep for no good reason. Sizes are
  now measured correctly.
- **Dropped connections no longer kill compaction.** An "aborted" reply
  without a real cancellation is now retried like any other transient error:
  the same summarizer call is repeated up to 3 times with growing pauses,
  and compaction continues from where it stopped. Only a real Esc/cancel
  stops it.

### Changed

- **Faster compaction on large models.** A summarizer with a big context window
  (e.g. 1M tokens) no longer has every chunk forced into several slow
  sequential calls; chunks are sized to respect the model's real headroom.
- **Retries that actually retry.** Transient provider failures now backtrack
  and retry up to 3 times with growing pauses, like pi does for its own
  requests. Quota and billing errors stop immediately.
- **Live progress.** While compacting, you see in the transcript which model
  runs, how many chunks there are, the current chunk and percent, split levels
  if any, and a final timing summary. None of it reaches the model context.
- **Compact model menu.** `/compact-model` now opens the same kind of list as
  `/model`: type to filter, ↑/↓ to pick, Enter/Esc — and it fits the screen.

### Added

- `DESIGN.md` now ships in the npm package, so the README link no longer 404s.
