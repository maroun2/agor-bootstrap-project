---
name: agor-session-chown
description: Change Agor session owner to a different user
triggers:
  - change session user
  - change session owner
  - create session as user
  - run session as
  - session chown
---

# Change Agor Session Owner

Transfer session ownership by updating the DB directly. The API doesn't support
creating sessions as another user, so we create → chown → prompt.

## Why create without prompt

If you pass `initialPrompt` to `agor_sessions_create`, the agent process starts
immediately under the creator's unix user. By creating empty first, you can change
ownership before any code runs — so the agent process spawns under the correct
unix user and the session shows up under the correct owner in the UI.

## Procedure

### 1. Find the target user

Call `agor_execute_tool(agor_users_list, {})`. You need two fields:
- `user_id` — UUID, goes into `created_by` column
- `unix_username` — OS user the agent process runs as

### 2. Create an empty session

```
agor_execute_tool(agor_sessions_create, {
  worktreeId: "<worktree-id>",
  agenticTool: "claude-code",
  title: "...",
  description: "..."
})
```

Do NOT pass `initialPrompt`. Note the returned `session_id`.

### 3. Update ownership in the database

```bash
sqlite3 ~/.agor/agor.db "UPDATE sessions \
  SET created_by = '<target-user-id>', \
      unix_username = '<target-unix-username>' \
  WHERE session_id = '<session-id>';"
```

### 4. Verify

```bash
sqlite3 ~/.agor/agor.db \
  "SELECT session_id, created_by, unix_username \
   FROM sessions WHERE session_id = '<session-id>';"
```

### 5. Send the initial prompt

```
agor_execute_tool(agor_sessions_prompt, {
  sessionId: "<session-id>",
  prompt: "Your full prompt here...",
  mode: "continue"
})
```

The session now runs under the target user's unix account and appears as
theirs in the Agor UI.

## Caveats

- **MCP token** is JWT-signed at creation time with the original user's ID.
  MCP tool calls inside the session may still authenticate as the creator.
  For most workflows (backtests, autoresearch) this doesn't matter.
- **Permissions** come from the session's config, not the new owner's defaults.
- **DB path** is `~/.agor/agor.db` by default.
- To **revert**, run the same UPDATE with the original user's ID.
