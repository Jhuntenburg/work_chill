# Work Chill Handoff and Status

Last updated: 2026-02-21 (America/New_York)

## Scope Completed

Implemented a low-friction macOS terminal workflow named `work_chill` with:

- `start` command to begin a day in one command.
- `start` now shows a "Welcome back" summary from the most recent `SHUTDOWN` event before `DAY_START` logging.
- Event logging now writes event-specific JSON keys (reduced log noise on non-shutdown events).
- Auto end-of-day trigger after 8 hours (or 2 minutes in test mode), with optional clock-time scheduling via `--end-at HH:MM`.
- Friction-check prompt loop every 55 minutes with AppleScript button UI (or terminal fallback).
- Pause/resume controls (`pause`, `resume`) so 8-hour shutdown is based on active work time (excluding paused breaks).
- Escalation flow after 3 consecutive `same_as_last` responses and on `stuck`.
- Guided shutdown ritual and meditation follow-up.
- Local state/log persistence under `~/.work_chill`.
- ISO-week log rotation (`Mon-Sun`) with compatibility symlink at `~/.work_chill/log.jsonl`.
- Idempotent shell setup in `~/.zshrc` with aliases.
- Health-check command: `work_chill doctor`.
- Prompt diagnostics and recovery logging in background mode.

## Files Added / Updated

- Script source in repo: `/Users/joseph/hss_docker/work_chill/work_chill`
- Installed executable: `/Users/joseph/bin/work_chill`
- Markdown viewer script (repo): `/Users/joseph/hss_docker/work_chill/chill_render_markdown`
- Markdown viewer script (legacy install): `/Users/joseph/bin/chill_render_markdown`
- Usage guide: `/Users/joseph/hss_docker/work_chill/work_chill_usage.md`
- This handoff file: `/Users/joseph/hss_docker/work_chill/storyDocs/work_chill_handoff_status.md`

## Shell Setup (`~/.zshrc`)

Confirmed lines:

- `export PATH="$HOME/bin:$PATH"`
- `alias wdstart='work_chill start'`
- `alias wdstatus='work_chill status'`
- `alias wdstop='work_chill cancel'`
- `alias wdshutdown='work_chill shutdown'`
- `alias wdtail='work_chill tail'`
- `alias chill='/Users/joseph/hss_docker/work_chill/chill_render_markdown'`

## Commands Implemented

- `work_chill start [--end-at HH:MM]`
- `work_chill pause`
- `work_chill resume`
- `work_chill status`
- `work_chill shutdown`
- `work_chill cancel`
- `work_chill debug-prompt`
- `work_chill set-meditation-url "<url>"`
- `work_chill clear-meditation-url`
- `work_chill tail`
- `work_chill doctor`
- `work_chill help`

## Data / State Layout

- Data dir: `/Users/joseph/.work_chill`
- State file: `/Users/joseph/.work_chill/state.sh`
  - `day_start_ts`
  - `prompt_pid`
  - `eod_pid`
  - `meditation_url`
  - `last_task_text`
  - `last_prompt_ts`
  - `paused` (`yes`/`no`)
  - `pause_started_ts`
  - `paused_total_secs`
  - `day_length_secs`
  - `prompt_interval_secs`
  - `consecutive_same_count`
  - `last_action`
- Weekly log directory: `/Users/joseph/.work_chill/logs`
- Current week event log: `/Users/joseph/.work_chill/logs/YYYY-Www.jsonl` (from `date +%G-W%V`)
- Compatibility symlink: `/Users/joseph/.work_chill/log.jsonl` -> current weekly file
- Legacy migrated file (one-time move when needed): `/Users/joseph/.work_chill/logs/legacy-log.jsonl`
- Background log: `/Users/joseph/.work_chill/bg.log`

## Current Runtime Status (captured 2026-02-18)

- `work_chill doctor` => `doctor_result: healthy`
- `work_chill status` during test run showed live workers:
  - `day_active: yes`
  - live `prompt_pid` and `eod_pid`
  - `last_prompt_at` updates after pulse execution

Installed binaries present and executable:

- `/Users/joseph/bin/work_chill`
- `/Users/joseph/hss_docker/work_chill/chill_render_markdown`

## Test Evidence Completed

Verified test-mode behavior (`WORK_CHILL_TEST=1`):

- Prompt loop fired repeatedly every ~10s and logged `TASK_PULSE_SKIPPED`.
- Auto shutdown executed and logged:
  - `{"event":"SHUTDOWN","action":"auto",...}` at `2026-02-18T03:21:42Z`
- Manual shutdown executed and logged:
  - `{"event":"SHUTDOWN","action":"manual",...}` at `2026-02-18T03:22:38Z`
- Meditation timer completion confirmed in background log:
  - `2026-02-18T03:22:53Z meditation_complete seconds=15`

Verified popup reliability patch and diagnostics:

- Reproduced original failure mode (before patch):
  - Prompt loop fired but `TASK_PULSE_SKIPPED` repeated with no visible prompt.
- Implemented and validated new prompt path with logging:
  - `bg.log` now includes per-attempt diagnostics, e.g.:
    - `2026-02-18T15:55:30Z prompt_dialog rc=0 output="Skip" stderr=""`
    - `2026-02-18T15:55:48Z prompt_dialog rc=0 output="Skip" stderr=""`
  - This confirms the dialog AppleScript executed successfully (exit code 0) in background pulses.
- Added command `work_chill debug-prompt` and validated it runs the same pulse code path:
  - Command output: `Debug prompt executed.`
  - `log.jsonl` appended `TASK_PULSE_SKIPPED` entries at matching timestamps.
- `work_chill doctor` re-run after patch:
  - `doctor_result: healthy`

Verified end-of-day clock-time scheduling:

- Started with `work_chill start --end-at 17:00`.
- Output confirmed scheduling:
  - `End-of-day scheduled for 17:00 (in 21120 seconds)`
- Background worker command included computed custom delay:
  - `bash /Users/joseph/bin/work_chill __eod-wait ... 21120`
- Invalid time format is rejected:
  - `work_chill start --end-at 5pm` -> `Invalid --end-at value: 5pm (expected HH:MM, 24-hour)`

## Important Behavior Notes

- Friction-check prompt now uses plain AppleScript `display dialog` + `activate` (not `System Events`) for better detached/background reliability.
- AppleScript dialog is constrained to 3 buttons (`Same as last`, `New task`, `Stuck`) due macOS `display dialog` limits; `Skip` is reached by timeout/cancel fallback path.
- Prompt attempts now log exit code/stdout/stderr to `~/.work_chill/bg.log`.
- On prompt-display failure, script sends a macOS notification and logs fallback reason instead of silently skipping.
- Shutdown dialogs still use AppleScript; terminal fallback is used when UI input fails.
- Timeouts were added for unattended reliability (especially for auto-shutdown path).
- Background jobs are spawned via `nohup` + `disown`.
- `start` is restart-safe: existing jobs are cancelled/replaced cleanly.
- `start --end-at HH:MM` uses local time; if the time has already passed today, it schedules for the next day.
- `wdstart` remains an alias to `work_chill start` and accepts args, e.g. `wdstart --end-at 17:00`.
- New startup context summary behavior:
  - On `work_chill start`, before `DAY_START`, script searches for latest `SHUTDOWN` in this order:
    - current week file (`~/.work_chill/logs/YYYY-Www.jsonl`)
    - previous weekly files (most recent 8 max)
    - `~/.work_chill/logs/legacy-log.jsonl` fallback
  - Extracted fields: `task_text`, `tomorrow_first_step`, `open_items`, `ts`.
  - Dialog title: `Welcome back`.
  - Message sections are shown only when each field is non-empty:
    - `Yesterday you closed with:`
    - `Next step you planned:`
    - `Open loop parked:`
  - If no prior `SHUTDOWN` exists, startup message is:
    - `Welcome - no prior shutdown context found.`
  - If `osascript` fails, startup prints terminal fallback and continues.
  - Parsing strategy:
    - Prefer `jq` when available.
    - Pure-shell fallback parser for environments without `jq`.
    - Lookup uses reverse scan (`tail -r`/`tac` fallback) to keep startup fast.
- Logging schema is now event-specific:
  - `DAY_START`: `ts`, `event`, `action`, `day_start_ts`
  - `TASK_PULSE`: `ts`, `event`, `action`, `task_text`, `day_start_ts`
  - `TASK_PULSE_SKIPPED`: `ts`, `event`, `action`, `day_start_ts`
  - `PAUSE`: `ts`, `event`, `action`, `day_start_ts`, `active_elapsed_secs`
  - `RESUME`: `ts`, `event`, `action`, `day_start_ts`, `paused_delta_secs`
  - `ESCALATION`: `ts`, `event`, `action`, `day_start_ts`, `choice`, plus optional `blocker_text`/`next_step_text`
  - `SHUTDOWN`: `ts`, `event`, `action`, `task_text`, `day_start_ts`, `tomorrow_first_step`, `open_items`
  - `CANCEL`: `ts`, `event`, `action`
- Backward compatibility retained:
  - Older log lines with extra keys are still valid input.
  - Welcome-back parser still reads latest `SHUTDOWN` context correctly.

## 2026-02-19 Change Validation

Ran requested verification flow after install:

1. `work_chill cancel`
2. `WORK_CHILL_TEST=1 work_chill start`
3. `work_chill status`
4. `tail -n 5 ~/.work_chill/log.jsonl`

Observed results:

- `start` succeeded and preserved existing behavior (timers active in test mode).
- `status` showed active day with live `prompt_pid` and `eod_pid`.
- Log format remained unchanged; latest entries include expected `CANCEL` and `DAY_START`.
- Terminal fallback path displayed the welcome summary when GUI invocation was unavailable in sandboxed run.

Additional logging-schema validation (same date):

1. `work_chill cancel`
2. `WORK_CHILL_TEST=1 work_chill start`
3. Triggered one `TASK_PULSE` (`new_task`) and one `TASK_PULSE_SKIPPED` (`skip`)
4. `work_chill shutdown` (manual)
5. Reviewed `tail -n 12 ~/.work_chill/log.jsonl`

Observed:

- New `TASK_PULSE` line excludes `tomorrow_first_step` and `open_items`.
- New `TASK_PULSE_SKIPPED` line contains only `ts`, `event`, `action`, `day_start_ts`.
- New `CANCEL` lines contain only `ts`, `event`, `action`.
- New `SHUTDOWN` line still contains `tomorrow_first_step` and `open_items`.

## 2026-02-20 Feature Upgrade Validation

Implemented requested upgrades in `/Users/joseph/hss_docker/work_chill/work_chill` and installed to `/Users/joseph/bin/work_chill`.

### Pause/Resume

- Added commands:
  - `work_chill pause`
  - `work_chill resume`
- `pause` behavior:
  - no active day -> informational exit
  - already paused -> informational exit
  - sets `paused=yes`, `pause_started_ts=now`
  - kills prompt + EOD workers safely
  - logs `PAUSE` with `active_elapsed_secs`
- `resume` behavior:
  - no active day / not paused -> informational exit
  - computes pause delta and increments `paused_total_secs`
  - clears pause state
  - reschedules prompt loop and EOD wait from remaining active seconds
  - logs `RESUME` with `paused_delta_secs`
- `status` now shows:
  - `paused`, `paused_total_secs`, `active_elapsed_secs`, `remaining_secs`
  - `day_length_secs`, `prompt_interval_secs`
  - paused marker (`timers: paused`) and no-live-pids state is expected while paused

### Escalation

- Added state tracking:
  - `consecutive_same_count`
  - `last_action`
- Triggering:
  - `same_as_last` increments count
  - non-`same_as_last` resets count
  - `stuck` triggers escalation immediately
  - `consecutive_same_count >= 3` triggers escalation
- Escalation UI:
  - title: `Quick reset`
  - buttons: `2-min break`, `Name blocker`, `Next micro-step`
  - timeout/cancel maps to ignore behavior
- Escalation actions:
  - break: opens `https://www.youtube.com/watch?v=Ih-rtou1PTc`, sends notifications, logs `ESCALATION` choice `break`
  - blocker: prompts one-sentence blocker, logs `ESCALATION` with `blocker_text`
  - next step: prompts smallest next step, logs `ESCALATION` with `next_step_text`
  - ignore: logs `ESCALATION` choice `ignore`
- Test-mode break timer:
  - `WORK_CHILL_TEST=1` -> break timer is 10 seconds

### Validation Run (2026-02-20)

Executed sequence:

1. `work_chill cancel`
2. `WORK_CHILL_TEST=1 work_chill start`
3. Simulated three same responses with:
   - `WORK_CHILL_TEST=1 WORK_CHILL_FORCE_ACTION=same_as_last work_chill debug-prompt` (x3)
4. Simulated stuck with:
   - `WORK_CHILL_TEST=1 WORK_CHILL_FORCE_ACTION=stuck work_chill debug-prompt`
5. `work_chill pause`
6. wait ~10s
7. `work_chill resume`
8. `work_chill status`
9. `tail -n 30 ~/.work_chill/log.jsonl`
10. `tail -n 40 ~/.work_chill/bg.log`

Observed:

- Escalation triggered after third same and on stuck:
  - bg log entries:
    - `escalation_trigger reason="same_as_last_3"`
    - `escalation_trigger reason="stuck"`
    - `escalation_dialog rc=0 output="Ignore" stderr=""` (dialog path executed)
- Pause/resume active-time accounting works:
  - `PAUSE` logged with `active_elapsed_secs:"15"`
  - `RESUME` logged with `paused_delta_secs:"10"`
  - status after resume showed:
    - `paused_total_secs: 10`
    - `active_elapsed_secs: 27`
    - `remaining_secs: 93`
    - live `prompt_pid` and `eod_pid`

## 2026-02-21 Weekly Log Rotation Upgrade

Implemented requested weekly log behavior in `/Users/joseph/hss_docker/work_chill/work_chill` with minimal, bash-only changes.

### What Changed

- Added ISO-week log target resolution per invocation:
  - uses `date +%G-W%V`
  - weekly file format: `~/.work_chill/logs/YYYY-Www.jsonl`
- Added `~/.work_chill/logs` auto-creation.
- Added backward-compat symlink management:
  - `~/.work_chill/log.jsonl` is now maintained as a symlink to current week file.
  - if a real file exists at `~/.work_chill/log.jsonl`, it is migrated once to `~/.work_chill/logs/legacy-log.jsonl` (append-safe when legacy already exists), then symlink is created.
- All log writes now go to weekly `LOG_FILE`.
- `work_chill tail` now tails current weekly log (via resolved `LOG_FILE`).
- `work_chill status` now prints current weekly path in `log_file:`.
- Welcome-back SHUTDOWN lookup now scans:
  - current week first,
  - then up to 8 most recent prior weekly logs,
  - then `legacy-log.jsonl` fallback.
- `work_chill doctor` now verifies the compatibility symlink.

### Validation Run (2026-02-21)

Executed sequence:

1. `work_chill doctor`
2. `work_chill status`
3. `WORK_CHILL_TEST=1 work_chill start`
4. `work_chill status`
5. `ls -la ~/.work_chill/logs | tail`
6. `readlink ~/.work_chill/log.jsonl`
7. `tail -n 5 ~/.work_chill/logs/$(date +%G-W%V).jsonl`
8. `work_chill cancel` (cleanup)

Observed:

- `doctor_result: healthy`
- Doctor confirms weekly file and symlink:
  - `[ok] log file writable: /Users/joseph/.work_chill/logs/2026-W08.jsonl`
  - `[ok] log symlink: /Users/joseph/.work_chill/log.jsonl -> /Users/joseph/.work_chill/logs/2026-W08.jsonl`
- `status` confirms:
  - `log_file: /Users/joseph/.work_chill/logs/2026-W08.jsonl`
- Logs directory proof:
  - `2026-W08.jsonl`
  - `legacy-log.jsonl`
- Symlink proof:
  - `readlink ~/.work_chill/log.jsonl` => `/Users/joseph/.work_chill/logs/2026-W08.jsonl`
- Last 5 log lines from current week file:
  - `{"ts":"2026-02-21T15:59:10Z","event":"CANCEL","action":"cancel"}`
  - `{"ts":"2026-02-21T16:01:16Z","event":"DAY_START","action":"start","day_start_ts":"1771689676"}`
  - `{"ts":"2026-02-21T16:01:16Z","event":"TASK_PULSE_SKIPPED","action":"skip","day_start_ts":"1771689676"}`
  - `{"ts":"2026-02-21T16:01:21Z","event":"TASK_PULSE","action":"new_task","task_text":"","day_start_ts":"1771689676"}`
  - `{"ts":"2026-02-21T16:01:26Z","event":"TASK_PULSE","action":"same_as_last","task_text":"","day_start_ts":"1771689676"}`

## Usage Quickstart

1. `source ~/.zshrc`
2. `work_chill doctor`
3. `work_chill start` (or `work_chill start --end-at 17:00`)
4. `work_chill status`
5. `work_chill shutdown` (manual) or wait for auto-shutdown
6. `chill` to open rendered usage guide in browser

## Revalidation Commands (future session)

```bash
export PATH="$HOME/bin:$PATH"
bash -n /Users/joseph/hss_docker/work_chill/work_chill
work_chill doctor
work_chill status
tail -n 20 /Users/joseph/.work_chill/log.jsonl
readlink /Users/joseph/.work_chill/log.jsonl
current_week="$(date +%G-W%V)"
ls -la /Users/joseph/.work_chill/logs | tail
tail -n 20 "/Users/joseph/.work_chill/logs/${current_week}.jsonl"
tail -n 20 /Users/joseph/.work_chill/bg.log
```

## Suggested Prompt for Next Conversation

Use this to resume quickly:

“Use `/Users/joseph/hss_docker/work_chill/storyDocs/work_chill_handoff_status.md` as the source of truth. Improve `work_chill` without breaking existing behavior. Keep install idempotent, preserve logs/state format, and re-run doctor/status validation after changes.”
