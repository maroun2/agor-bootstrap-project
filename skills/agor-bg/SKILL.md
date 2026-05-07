---
name: agor-bg
description: Run long-running commands in the background and get notified when they finish. Use this skill when the user wants to run a process that takes a long time (builds, trading bots, test suites, data pipelines) without blocking the current session. Also use when the user mentions "background task", "nohup", "run in background", "notify when done", "agor-bg", "agor-prompt", or wants to send a prompt to another Agor session programmatically.
---

# agor-bg — Background Tasks with Session Notification

Two CLI tools at `~/.local/bin/` for running background tasks and notifying Agor sessions.

## Tools

### `agor-bg` — Run command in background, notify on completion

Runs a command via `nohup` in the background. When the command finishes, it automatically sends a prompt to the specified Agor session with the exit code and last 20 lines of output.

```bash
agor-bg <session-id> <command> [args...]
```

**Examples:**

```bash
agor-bg 278c8b53 .venv/bin/python3 -m live.runner --auto
agor-bg 278c8b53 make test
agor-bg 278c8b53 sleep 300
```

**What happens:**
1. Command starts in background (survives session death)
2. Output goes to `/tmp/agor-bg-<timestamp>.log`
3. On completion, sends prompt to session with exit code + last 20 lines
4. If session is busy, prompt gets queued and runs when session becomes idle

### `agor-prompt` — Send a prompt to any Agor session

Sends a prompt to an Agor session via the daemon REST API (`POST /sessions/:id/prompt`). Works whether the target session is idle (executes immediately) or busy (gets queued).

```bash
agor-prompt <session-id> <prompt>
```

**Examples:**

```bash
agor-prompt 278c8b53 "Check the test results in /tmp/test.log"
agor-prompt 278c8b53 "Deploy is done, verify staging"
```

## How to use from within a session

To find the current session ID, use the `agor_sessions_get_current` MCP tool, then:

```bash
agor-bg SESSION_ID command args...
```

**NOTE:** `~/.local/bin` must be on `PATH`. It is added via `~/.local/bin/env` which is sourced by `.profile` and `.zshrc`, but may not be present in sudo-spawned shells. In that case use full path: `~/.local/bin/agor-bg`.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `AGOR_DAEMON_URL` | `http://localhost:3031` | Agor daemon URL |
| `AGOR_TOKEN` | (built-in default) | JWT auth token for local daemon |
| `AGOR_BG_LOGDIR` | `/tmp` | Directory for log files |

## When to use

- Long-running processes (trading bots, builds, test suites, data pipelines)
- Tasks that might outlive the current session
- Fire-and-forget with guaranteed notification
- Chaining sessions: finish task in one, notify another
- Waking up an idle session from cron or external script

## Important notes
- Background process survives even if the Agor session dies
- Logs always written to disk regardless of notification success
- If session ID is wrong or daemon is down, command still runs — only notification fails
- Token is permanent (no expiry), works for any session owned by the user
- Token is derived from daemon `jwtSecret` — rotates if secret rotates
