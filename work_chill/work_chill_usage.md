# Work Chill Usage Guide

This guide is for:
- `/Users/joseph/bin/work_chill`
- `/Users/joseph/hss_docker/work_chill/chill_render_markdown` (used by `chill`)

## One-time setup

1. Open a new terminal (or run `source ~/.zshrc`) so PATH and aliases are loaded.
2. Verify health:

```bash
work_chill doctor
```

Expected: `doctor_result: healthy`

## Daily use

1. Start your day:

```bash
work_chill start
```

Or start with a fixed end time:

```bash
work_chill start --end-at 17:00
```

2. Friction check appears every 55 minutes:
- `Same as last`
- `New task`
- `Stuck`

Timeout/cancel is treated as `Skip`.

3. If you take a substantial break:

```bash
work_chill pause
```

Then resume:

```bash
work_chill resume
```

`pause/resume` keeps the day target based on active time (paused time is excluded).

4. End of day:
- automatic after active target time
- or manual anytime:

```bash
work_chill shutdown
```

## Escalation behavior

- `Stuck` always triggers a Quick reset dialog.
- `Same as last` 3 times in a row triggers Quick reset.
- Quick reset options:
  - `2-min break`
  - `Name blocker`
  - `Next micro-step`
  - timeout/cancel -> ignore

## Commands

```bash
work_chill start [--end-at HH:MM]
work_chill pause
work_chill resume
work_chill status
work_chill shutdown
work_chill cancel
work_chill debug-prompt
work_chill set-meditation-url "<url>"
work_chill clear-meditation-url
work_chill tail
work_chill doctor
work_chill help
```

## Aliases

```bash
wdstart      # work_chill start (accepts args)
wdstatus     # work_chill status
wdstop       # work_chill cancel
wdshutdown   # work_chill shutdown
wdtail       # work_chill tail
chill        # opens this guide in browser
```

## Status fields

`work_chill status` includes:
- `paused`
- `paused_total_secs`
- `active_elapsed_secs`
- `remaining_secs`
- prompt/eod PID health

## Files and logs

- State file: `~/.work_chill/state.sh`
- Event log: `~/.work_chill/log.jsonl`
- Background log: `~/.work_chill/bg.log`
- Guide file: `/Users/joseph/hss_docker/work_chill/work_chill_usage.md`

## Test mode

```bash
WORK_CHILL_TEST=1 work_chill start
```

In test mode:
- prompt interval = 10 seconds
- day length = 2 minutes (unless `--end-at` is provided)
- meditation timer = 15 seconds
- escalation break timer = 10 seconds
