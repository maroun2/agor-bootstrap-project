---
name: board-context
description: Generate a deterministic Markdown board context snapshot from an Agor board. Output to stdout. Covers zones (with trigger previews), worktrees (notes + rules previews, inline wt_id), key sessions (last-msg tail, inline sess_id), open PRs, project docs deduplicated by hash, and local context.
triggers:
  - board context
  - board-context
  - agor board context
  - board snapshot
  - board summary
  - user wants to work on a project
  - user wants to start or continue work on a board
  - user wants to coordinate or orchestrate a project
  - user asks what is the state of a project
  - user does not know which session to talk to
  - user is not sure what is happening on a board
  - user wants to pick up where they left off on a project
  - general orientation before starting work on any Agor project
argument-hint: "<board-slug-or-id>"
---

# Board Context — Deterministic Agor Board Snapshot

Generate a compact Markdown snapshot of an Agor board for use as starter
context in a new session. Output goes to stdout — pipe or redirect as needed.
Queries `~/.agor/agor.db` directly — no MCP, no LLM, no network calls (PR
detection via `gh` is best-effort and optional).

## When to use

Run this skill **before starting or resuming any project work** on an Agor board:

- You (or the user) don't know which session to continue in
- You need to orient yourself on what worktrees exist and their current state
- User says "let's work on X" or "continue on X" and you don't have context yet
- You need to figure out what's running, what's idle, what's in which zone
- Coordinator session starting up and needs a full picture before acting

## Usage

```bash
agor-board-context <board-slug-or-id>
agor-board-context my-project > board-context.md
```

Script: `.claude/skills/board-context/scripts/agor-board-context`

## Requirements

- `sqlite3`
- `jq`
- Agor installed (`~/.agor/agor.db` must exist)
- `gh` CLI (optional — for open PR detection)

## Output sections

| Section | Content |
|---------|---------|
| Board header | Name, description, counts, shared root path, board ID |
| Zones | Trigger, agent, first-line template preview + `…` |
| Worktrees | Name, repo, branch, zone, sessions, latest, notes preview, rules preview, `wt_id` |
| Key sessions | Running + long-lived (≤8), last-msg tail, `sess_id` |
| Pull Requests | Notes-regex + `gh pr list` (open), state shown |
| Project Docs | roadmap/spec/doc files, deduplicated by sha256 hash, relative paths |
| Local Context | CLAUDE.md hash, skills dir+count, rules dir+count |

Each section ends with an inline `>` MCP call template. No detached drill-down appendix.

## Environment

| Var | Default | Description |
|-----|---------|-------------|
| `AGOR_DB` | `~/.agor/agor.db` | Override database path |
