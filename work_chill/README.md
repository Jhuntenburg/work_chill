# work_chill

Personal macOS work pacing helper script.

## What it does

- Starts a work session with `work_chill start`
- Shows a "Welcome back" summary from the latest `SHUTDOWN` context
- Runs friction-check prompts every 55 minutes (10s in test mode)
- Supports `pause` / `resume` so the day target is based on active work time
- Triggers escalation on `stuck` and after 3 consecutive `same_as_last` responses
- Supports manual and automatic shutdown flow
- Logs events to weekly JSONL files at `~/.work_chill/logs/YYYY-Www.jsonl` (ISO week)
- Maintains `~/.work_chill/log.jsonl` as a symlink to the current week file for compatibility

## Script in this repo

- `work_chill`
- `chill_render_markdown`
- `work_chill_usage.md`
- `storyDocs/work_chill_handoff_status.md`

## Install

```bash
cd /Users/joseph/hss_docker/work_chill
install -m 755 ./work_chill "$HOME/bin/work_chill"
```

Make sure `~/bin` is on `PATH`.

## Common commands

```bash
work_chill start
work_chill start --end-at 17:00
work_chill pause
work_chill resume
work_chill status
work_chill shutdown
work_chill cancel
work_chill debug-prompt
work_chill doctor
```

## Data layout

- State: `~/.work_chill/state.sh`
- Weekly logs: `~/.work_chill/logs/YYYY-Www.jsonl`
- Compatibility link: `~/.work_chill/log.jsonl` -> current weekly file
- Background log: `~/.work_chill/bg.log`

## Quick reference (`chill`)

Guide source now lives in-repo at:
- `/Users/joseph/hss_docker/work_chill/work_chill_usage.md`

Renderer script in repo:
- `/Users/joseph/hss_docker/work_chill/chill_render_markdown`

Recommended alias:

```bash
alias chill='/Users/joseph/hss_docker/work_chill/chill_render_markdown'
```

## Test mode

```bash
WORK_CHILL_TEST=1 work_chill start
```

Test mode values:
- prompt interval: `10` seconds
- day length: `120` seconds (unless `--end-at` is provided)
- meditation timer: `15` seconds
- escalation break timer: `10` seconds
