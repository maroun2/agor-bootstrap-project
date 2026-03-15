# Roadmap — Agor Bootstrap Project

A reusable template for bootstrapping autonomous multi-agent development pipelines in [Agor](https://agor.live). Any Agor user can fork this template and go from zero to a fully operational Plan > Implement > Review > Test > Done pipeline in minutes.

## Tech Stack

- **Platform:** Agor (git worktrees, zone triggers, MCP tools)
- **Agents:** Claude Code (primary), with zone templates compatible with any Agor-supported agent
- **Configuration:** `board.custom_context` for project-specific settings (test commands, build commands, etc.)
- **Docs:** Markdown files in `doc/`, `CLAUDE.md` as agent convention hub

## Architecture

- **Coordinator worktree** (persistent, outside zones) — orchestration only, never implements
- **Zone-triggered agent sessions** — specialized prompts for Plan, In Progress, Review, Test, Done
- **`test-main` worktree** (persistent, outside zones) — runs e2e tests for all feature branches
- **Feature worktrees** — flow through the pipeline autonomously

### Configuration via `board.custom_context`

Project-specific commands and settings are stored in `board.custom_context` so zone templates can reference them without hardcoding. Expected fields:

| Field | Description | Example |
|-------|-------------|---------|
| `build_command` | Command to build the project | `npm run build` |
| `test_command` | Command to run unit tests | `npm test` |
| `e2e_test_command` | Command to run e2e tests | `npm run test:e2e` |
| `run_command` | Command to start the application | `npm start` |

Zone templates reference these as `{{ board.custom_context.build_command }}`, etc.

## Phase 1: MVP — Working Template

Make the existing template fully functional and project-agnostic.

| # | Task | Description | Parallel? |
|---|------|-------------|-----------|
| 1 | Generalize zone templates | Replace Godot-specific commands in zone trigger templates with `board.custom_context` references (`build_command`, `test_command`, `e2e_test_command`, `run_command`) | Yes (with #2) |
| 2 | Document custom_context schema | Create documentation for the expected `board.custom_context` fields that zone templates reference, with examples for different project types | Yes (with #1) |
| 3 | Update board-setup.md | Add `custom_context` setup instructions to the board setup documentation so users know to configure it | After #2 |
| 4 | End-to-end bootstrap test | Run the full bootstrap flow on a fresh project to verify all steps complete successfully and all artifacts are created | After #1, #2, #3 |

### Definition of Done (Phase 1)

- All zone templates use `board.custom_context` instead of hardcoded commands
- `board.custom_context` schema is documented with examples
- `board-setup.md` includes custom_context configuration
- Bootstrap flow completes successfully on a fresh project

## Phase 2: Polish & Examples

- Step-by-step tutorial with screenshots
- Example `board.custom_context` configurations for common project types:
  - Node.js web app
  - Python CLI
  - Godot game
  - Go service
- Troubleshooting guide for common issues
- README refinements based on user feedback

## Testing Strategy

- **Bootstrap flow test:** Run the coordinator bootstrap on a fresh repo and verify all artifacts are created (CLAUDE.md, roadmap.md, api.md, zones, worktrees)
- **Zone trigger test:** Move a worktree through each zone and verify the correct agent prompt is generated with resolved template variables
- **Template variable test:** Verify all `{{ }}` variables resolve correctly in zone prompts, including `board.custom_context` fields
- **Acceptance criteria format:** Each task worktree includes AC in its notes; the review agent checks against them

## Work Breakdown Summary

**Can run in parallel:**
- Task #1 (Generalize zone templates) and Task #2 (Document custom_context schema)

**Sequential dependencies:**
- Task #3 (Update board-setup.md) depends on Task #2
- Task #4 (End-to-end bootstrap test) depends on all other tasks
