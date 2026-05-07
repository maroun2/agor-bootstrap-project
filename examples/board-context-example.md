# 🤖 Agor

> 2026-05-07T08:19:32Z | agor-board-context v0.3.0  
> Template project for bootstrapping new Agor projects

**3** worktrees · **12** sessions · **2** active

## Coordinator (likely)

**Worktree:** add-iphone-native-app  
**Session:** Bugs and features iOS (claude-code, running)  
**CLAUDE.md:** `/home/USER/.agor/worktrees/USER/agor/add-iphone-native-app/CLAUDE.md`  

## Zones

| Zone | Trigger | Agent | Template preview |
|------|---------|-------|------------------|
| Plan | always_new | claude-code | You are a planning agent on the {{ board.name }} board. |
| Done | always_new | claude-code | The task and worktree are complete and have been tested. Create a Pull Request i… |
| In Progress | always_new | claude-code | You are an implementation agent on the {{ board.name }} board. |
| Review | always_new | claude-code | You are a code review agent on the {{ board.name }} board. |
| Test | always_new | claude-code | You are a test agent on the {{ board.name }} board. |

## Worktrees

| Name | Repo | Branch | Zone | Sessions | Latest | Notes |
|------|------|--------|------|----------|--------|-------|
| vps-nix | USER/vps-nix | main | — | 5 | setup vps server (idle) | **Server:** RackNerd VPS (racknerd) |
| add-iphone-native-app | USER/agor | add-iphone-native-app | — | 4 | Bugs and features iOS (running) |  |
| board-context-cli | USER/agor-bootstrap-project | board-context-cli | Plan | 3 | New direction for agor-board-context. Replace the curre… (running) | Implement a deterministic CLI/script in the agor-bootstrap-p… |

## Key Sessions

| Session | Worktree | Status | Kind |
|---------|----------|--------|------|
| setup vps server | vps-nix | idle | 1165h |
| New direction for agor-board-context. Replace the curre… | board-context-cli | running | running |
| Whisper | add-iphone-native-app | idle | 388h |
| Bugs and features iOS | add-iphone-native-app | running | running |
| disk | vps-nix | idle | 724h |
| Oracle Cloud VPS setup | vps-nix | idle | 695h |
| iOS - Implemnet - Codex | add-iphone-native-app | idle | 57h |

## Local Context

| Worktree | CLAUDE.md | Skills | Rules |
|----------|-----------|--------|-------|
| vps-nix | yes | — | — |
| add-iphone-native-app | yes | — | — |
| board-context-cli | — | 5 | — |

## Drill-down

Source-of-truth MCP calls and file paths — use these when you need detail beyond this summary.

### Board
```
agor_boards_get(boardId="fa1d135e-32e4-ade2-5837-c350f43bb1ce")
```

### Worktrees
```
agor_worktrees_list(repoId="3ecfb107-7d78-414a-a3ee-c700174d910b", includeArchived=false)  # repo: USER/agor
```

- **add-iphone-native-app**: `agor_worktrees_get(worktreeId="36486fc6-7dd9-4149-92d7-db691fce6fb9")`

```
agor_worktrees_list(repoId="57ab55f9-401f-48fa-be7f-916d741e62e9", includeArchived=false)  # repo: USER/agor-bootstrap-project
```

- **board-context-cli**: `agor_worktrees_get(worktreeId="1e24aab4-8f5c-4aea-ab43-2d78bffd3bbc")`

```
agor_worktrees_list(repoId="ddb55d98-700f-4f2b-8758-f17f0ddc7f93", includeArchived=false)  # repo: USER/vps-nix
```

- **vps-nix**: `agor_worktrees_get(worktreeId="fbefa669-f0b9-44de-8a55-0f4553c32765")`


### Key Sessions
- **setup vps server** (idle, vps-nix): `agor_sessions_get(sessionId="0380b0cd-ce9d-4dec-9f7f-bd0a5f2b1df6")`
- **New direction for agor-board-context. Replace the curre…** (running, board-context-cli): `agor_sessions_get(sessionId="08265238-95cf-4b8a-85a5-9001117d81cb")`
- **Whisper** (idle, add-iphone-native-app): `agor_sessions_get(sessionId="0a1fe83a-93f2-46ba-94db-8f8ac22d7664")`
- **Bugs and features iOS** (running, add-iphone-native-app): `agor_sessions_get(sessionId="3a53eee5-4ab6-416a-a0d0-ea47d4a262de")`
- **disk** (idle, vps-nix): `agor_sessions_get(sessionId="9d1d7f5a-22d0-4c99-8538-d234589e2d06")`
- **Oracle Cloud VPS setup** (idle, vps-nix): `agor_sessions_get(sessionId="e18ad3df-4001-4bfd-8d02-ee444d3da14d")`
- **iOS - Implemnet - Codex** (idle, add-iphone-native-app): `agor_sessions_get(sessionId="e5f5185a-c3f8-4969-93e4-40883cc0d9a5")`

```
# Messages for "setup vps server"
agor_messages_list(sessionId="0380b0cd-ce9d-4dec-9f7f-bd0a5f2b1df6", contentMode="full", includeToolCalls=false, limit=50)

# Tasks for "setup vps server"
agor_tasks_list(sessionId="0380b0cd-ce9d-4dec-9f7f-bd0a5f2b1df6", limit=20)
agor_tasks_get(taskId="<task_id>")
```

### Local Files
- vps-nix CLAUDE.md: `/home/USER/.agor/worktrees/USER/vps-nix/vps-nix/CLAUDE.md`
- add-iphone-native-app CLAUDE.md: `/home/USER/.agor/worktrees/USER/agor/add-iphone-native-app/CLAUDE.md`
- board-context-cli skills:   `/home/USER/.agor/worktrees/USER/agor-bootstrap-project/board-context-cli/.claude/skills`

