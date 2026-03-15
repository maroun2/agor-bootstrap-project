# API Documentation — Agor Bootstrap Project

This document defines the interface contract between agents in the bootstrap pipeline and the customization points for adapting the template to different projects.

## 1. Board Custom Context Schema

Project-specific configuration is stored in `board.custom_context`. Zone templates reference these values so agents know how to build, test, and run the project.

### Required Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `build_command` | string | Command to build the project | `npm run build` |
| `test_command` | string | Command to run unit tests | `npm test` |
| `e2e_test_command` | string | Command to run end-to-end tests | `npm run test:e2e` |
| `run_command` | string | Command to start the application for testing | `npm start` |

### Optional Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `lint_command` | string | Command to run linting | `npm run lint` |
| `project_type` | string | Hint for agents about the project type | `node`, `python`, `godot` |

### Example Configurations

**Node.js Web App:**
```json
{
  "build_command": "npm run build",
  "test_command": "npm test",
  "e2e_test_command": "npm run test:e2e",
  "run_command": "npm start",
  "lint_command": "npm run lint",
  "project_type": "node"
}
```

**Python CLI:**
```json
{
  "build_command": "pip install -e .",
  "test_command": "pytest tests/unit",
  "e2e_test_command": "pytest tests/e2e",
  "run_command": "python -m myapp",
  "lint_command": "ruff check .",
  "project_type": "python"
}
```

**Godot Game:**
```json
{
  "build_command": "godot --headless --export-release",
  "test_command": "godot --headless --script addons/gut/gut_cmdln.gd",
  "e2e_test_command": "godot --headless -- --bot-api",
  "run_command": "godot --headless -- --bot-api",
  "project_type": "godot"
}
```

## 2. Template Variables

Zone trigger templates use Mustache-style `{{ }}` variables. These are resolved by Agor when a session is created.

### Available Variables

| Variable | Scope | Description |
|----------|-------|-------------|
| `{{ worktree.name }}` | Worktree | Worktree slug name |
| `{{ worktree.ref }}` | Worktree | Git branch or tag |
| `{{ worktree.issue_url }}` | Worktree | Associated GitHub issue URL |
| `{{ worktree.pull_request_url }}` | Worktree | Associated PR URL |
| `{{ worktree.notes }}` | Worktree | Freeform markdown notes |
| `{{ worktree.custom_context }}` | Worktree | Custom context object |
| `{{ board.name }}` | Board | Board display name |
| `{{ board.custom_context }}` | Board | Board-level custom context (see Section 1) |
| `{{ zone.label }}` | Zone | Zone display name (e.g., "Plan") |
| `{{ zone.status }}` | Zone | Zone status |

**Not available in zone triggers:** `session.*`, `worktree.path`, `repo.*`. These are only available in system prompt or environment templates.

## 3. Zone Workflow Contract

Worktrees flow through zones in this order: **Plan → In Progress → Review → Test → Done**

### Zone Responsibilities

| Zone | Agent Role | Moves Worktree? | Reads | Writes |
|------|-----------|-----------------|-------|--------|
| **Plan** | Planning | → In Progress (after user confirms) | CLAUDE.md, worktree notes, issue | Appends plan to worktree notes |
| **In Progress** | Implementation | → Review (once building) | CLAUDE.md, worktree notes | Code, commits, pushes, appends summary to notes |
| **Review** | Code review | → Test (if clean) | Diff, worktree notes (AC) | Nothing (review only) |
| **Test** | Testing | → Done (if all pass) | CLAUDE.md, test output | Nothing (test only) |
| **Done** | PR creation | None | Worktree notes, issue/PR | Creates PR, updates worktree PR URL |

### Zone Transition Rules

1. **Plan → In Progress:** Only after user explicitly confirms the plan
2. **In Progress → Review:** Only after the project builds without errors
3. **Review → Test:** Only if no blocking issues found
4. **Test → Done:** Only after receiving the test-main e2e report AND all tests pass
5. **Review → In Progress (feedback):** Via `agor_sessions_prompt`, not zone move
6. **Test → In Progress (feedback):** Via `agor_sessions_prompt`, not zone move

## 4. Inter-Agent Communication

Agents communicate via `agor_sessions_prompt`. Sessions are discoverable by name convention.

### Session Naming Convention

Zone-triggered sessions are automatically named: `Session from zone "<Zone Label>"`

Examples:
- `Session from zone "Plan"` — the planning session for a worktree
- `Session from zone "In Progress"` — the implementation session
- `Session from zone "Review"` — the review session
- `Session from zone "Test"` — the test session

### Communication Patterns

**Review feedback loop:**
1. Review agent finds issues
2. Review agent sends feedback via `agor_sessions_prompt` to `Session from zone "In Progress"`
3. Implementation agent fixes issues, commits, pushes
4. Implementation agent prompts back `Session from zone "Review"` with fix summary
5. Review agent re-reviews

**Test feedback loop:**
1. Test agent finds failures
2. Test agent sends failure report via `agor_sessions_prompt` to `Session from zone "In Progress"`
3. Implementation agent fixes, commits, pushes
4. Implementation agent prompts back `Session from zone "Test"` with fix summary
5. Test agent re-runs tests

**E2E test delegation:**
1. Test zone agent creates session named `Test: <worktree.name>` on `test-main` worktree
2. Test-main agent runs e2e tests against the feature branch
3. Test-main agent reports results back via `agor_sessions_prompt` to `Session from zone "Test"`

## 5. Worktree Notes Format

Worktree notes are the primary way to pass context between zones. Notes are **append-only** — agents add to them but do not remove prior content.

### Initial Notes (set by coordinator)

```markdown
# <Task Name>

## Description
<What this task does>

## Acceptance Criteria
- [ ] <Criterion 1>
- [ ] <Criterion 2>

## API References
- See `api.md` section X

## Dependencies
- Depends on: <other worktree names, or "none">
- Blocks: <other worktree names, or "none">
```

### After Planning

```markdown
## Implementation Plan
1. Step 1
2. Step 2
   - Substep 2a (can be parallelized with 2b)
   - Substep 2b
3. Step 3

### Risks
- <Risk description>

### Tests Needed
- <Test description>
```

### After Implementation

```markdown
## Implementation Summary
- <What was done>
- <Key decisions made>
- Branch pushed: `<branch-name>`
```

## 6. Customization Guide

### Adding a New Zone

1. Use `agor_boards_update` with `upsertObjects` to add the zone
2. Define `x`, `y`, `width`, `height` for positioning
3. Set `trigger.behavior` to `always_new` for automatic sessions
4. Set `trigger.agent` to the agent type (e.g., `claude-code`)
5. Write the `trigger.template` prompt referencing available template variables

### Modifying Zone Prompts

Update zone templates via `agor_boards_update` with `upsertObjects`, using the same zone ID to overwrite.

### Changing the Pipeline Order

The pipeline order is enforced by zone prompt templates (each zone knows which zone to move to next). To change the order:
1. Update the "move to" instructions in each affected zone's template
2. Update the session naming conventions in the "Finding Sessions" sections
3. Update this document's Zone Workflow Contract section

### Skipping Zones

To skip a zone (e.g., skip Review for small changes), the preceding zone's agent should move the worktree directly to the next zone in the chain. This requires modifying the zone template.

## 7. Persistent Worktrees

Two worktrees live permanently outside the zones:

| Worktree | Purpose | Location |
|----------|---------|----------|
| `coordinator` | Orchestrates the project, creates worktrees, manages docs | Top of board, above zones |
| `test-main` | Runs e2e tests against feature branches on request | Side of board, outside zones |

These are never moved into zones after initial setup.
