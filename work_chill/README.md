# work_chill

Personal macOS work pacing helper script.

## What it does

- Starts a work session with `work_chill start`
- Shows a "Welcome back" summary from the latest `SHUTDOWN` log context
- Runs periodic friction-check prompts
- Supports manual and automatic shutdown flow
- Logs events to `~/.work_chill/log.jsonl`

## Script in this repo

- `work_chill`
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
work_chill status
work_chill shutdown
work_chill cancel
work_chill doctor
```

## Test mode

```bash
WORK_CHILL_TEST=1 work_chill start
```

Test mode uses short intervals for faster validation.
