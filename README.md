# Agor Bootstrap Project

A reusable template for setting up a [dark factory](https://simonwillison.net/2026/Jan/28/the-five-levels/) style software development pipeline in [Agor](https://agor.live). Targets **Level 4-5** AI orchestration — where AI agents handle implementation, testing, and review autonomously, while humans focus on specs, direction, and validation.

Inspired by the [five levels of AI-assisted programming](https://hackernoon.com/the-dark-factory-pattern-moving-from-ai-assisted-to-fully-autonomous-coding): from spicy autocomplete (L1) to the dark factory (L5) where specs go in and software comes out. This template sets up the coordinator, workflow zones, and agent prompts to get there.

## Features

- **Isolated test agent** — the test agent lives in its own worktree with no access to implementation code, testing purely against the API specification. This ensures tests validate behavior, not implementation details
- **Independent agent-worktrees** — each task runs in its own git worktree with a dedicated agent session, enabling safe parallel development with full code isolation
- **API-first design** — API documentation is finalized and agreed upon before any implementation begins, serving as the single source of truth for both developers and the test agent
- **Zone-based workflow** — worktrees flow through Plan → In Progress → Review → Test → Done with auto-triggered agent sessions
- **Autonomous pipeline** — agents plan, implement, review, and test without human intervention
- **Persistent coordinator** — orchestrator agent oversees the full lifecycle from concept to release
- **Interactive roadmap creation** — market research, concept refinement, and phased planning with the user
- **Resumable bootstrap** — coordinator checks existing artifacts and skips completed steps on restart
- **Reusable template** — works for any project type; just create a board and point the coordinator at this repo

## Prerequisites

- [Agor](https://agor.live) installed and running
- `gh` CLI authenticated (`gh auth status`)
- Git configured (`user.name`, `user.email`)

## How to Start

### Option A: New project (no board yet)

In any Agor session, tell the agent about your new project — name, description, what it does. Once you have that context established, give it this prompt:

```
Create a new Agor project:

1. Create a GitHub repo: `gh repo create <org>/<name> --public --description "<description>"`
   - Create initial commit and push to main
   - Register in Agor: `agor_repos_create_remote`

2. Backup the Agor database: `cp ~/.agor/agor.db ~/.agor/agor.db.backup-$(date +%Y%m%d-%H%M%S)`

3. Create a board via SQLite (`~/.agor/agor.db`):
   - Get user ID from `agor_users_get_current`
   - INSERT INTO boards (board_id, created_at, updated_at, created_by, name, slug, data)
     VALUES (
       lower(hex(randomblob(4)) || '-' || hex(randomblob(2)) || '-' || hex(randomblob(2)) || '-' || hex(randomblob(2)) || '-' || hex(randomblob(6))),
       strftime('%s', 'now') * 1000,
       strftime('%s', 'now') * 1000,
       '<user_id>',
       '<Project Name>',
       '<project-slug>',
       json('{"description":"<description>","icon":"<emoji>"}')
     );
   - Verify with `agor_boards_get`

4. Create coordinator worktree on the board
   - Name: `coordinator`
   - Set worktree notes with project context
   - Create session titled "Project bootstrap from maroun2/agor-bootstrap-project" with prompt:

You are the project coordinator for {{ board.name }}.

## Context
- **Notes:** {{ worktree.notes }}
- **Board context:** {{ board.custom_context }}

## Your Role
You oversee the entire project from concept to release. You do NOT implement —
you research, plan, discuss with the user, and orchestrate the project setup.

## Setup
1. Fetch all files from https://github.com/maroun2/agor-bootstrap-project/tree/main/doc
2. Save them to your project's doc/ directory
3. Read doc/bootstrap.md and follow the instructions step by step
```

### Option B: Existing board

If you already have a board and repo, just create the coordinator worktree and start a session:

#### Short prompt
```
Fetch all files from https://github.com/maroun2/agor-bootstrap-project/tree/main/doc
into your project's doc/ directory. Then read doc/bootstrap.md and follow the instructions.
```

## What the Coordinator Does

The coordinator handles the full bootstrap in order:

| Step | Doc | What happens |
|------|-----|--------------|
| 0 | `board-setup.md` | Set up board zones, create coordinator worktree if missing |
| 1 | `create-roadmap.md` | Market research, concept refinement, roadmap |
| 2 | `create-api-doc.md` | API documentation |
| 3 | `create-tests.md` | Test agent setup |
| 4 | `create-phase1.md` | Phase 1 task worktrees |

## Documentation

| File | Description |
|------|-------------|
| `doc/bootstrap.md` | Step-by-step process for the coordinator |
| `doc/board-setup.md` | Zone layout and coordinator zone definition |
| `doc/create-roadmap.md` | Market research, concept refinement, roadmap creation |
| `doc/create-api-doc.md` | API documentation drafting and review |
| `doc/create-tests.md` | Test agent setup and placement |
| `doc/create-phase1.md` | Phase 1 task worktree creation |

## Workflow

```
Coordinator (outside zones, oversees everything)
        |
        v
  0. Set up board zones (if not already done)
  1. Create roadmap (interactive with user)
  2. Create API docs (interactive with user)
  3. Set up test agent (runs through pipeline, then lives outside zones)
  4. Create Phase 1 worktrees (placed in Plan zone)
        |
        v
  [ Plan ] -> [ In Progress ] -> [ Review ] -> [ Test ] -> [ Done ]
```
