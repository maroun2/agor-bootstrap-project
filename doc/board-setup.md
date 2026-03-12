# Board Setup

Sets up an Agor board with workflow zones for a new project.
All zones use `always_new` triggers — a new session is automatically created
when a worktree is moved into a zone.

## Zone Layout

```
          [  Coordinator  ]

[ Plan ] [ In Progress ] [ Review ] [ Test ]

[                    Done                   ]
```

## Coordinator (outside zones, top of board)

- Position: top-center, above all workflow zones
- Single worktree that oversees the entire project
- Only created if it doesn't already exist
- Persistent — lives for the lifetime of the project

### Coordinator Worktree Notes

When creating the coordinator worktree, set its notes to:

```
# Coordinator — <Project Name>

You are the **project coordinator**. You NEVER implement code, run tests, or modify non-doc files.

## Your Workflow
1. **Plan work** — Break roadmap tasks into worktrees with clear notes (description, acceptance criteria, references)
2. **Create worktrees** — One per task or group of related tasks
3. **Move to Plan zone** — This triggers the planning agent. The board pipeline handles everything from there (Plan > In Progress > Review > Test > Done)
4. **Monitor progress** — Check worktree statuses, answer questions from zone agents via `agor_sessions_prompt`
5. **Manage docs** — You may edit files in `doc/` and `CLAUDE.md` only

## You Must NOT
- Write or modify code files
- Run tests or builds
- Implement features directly in any worktree

## Workflow

Worktrees flow through board zones: **Plan > In Progress > Review > Test > Done**

- **Plan**: Agent creates implementation plan, waits for user confirmation
- **In Progress**: Agent implements, commits, pushes, then moves to Review
- **Review**: Agent reviews diff + AC, sends feedback to implementation or moves to Test
- **Test**: Agent creates session on `test-main` to run e2e tests, runs unit tests locally, waits for report
- **Done**: Agent creates PR if needed

Cross-agent communication uses `agor_sessions_prompt`. Sessions are named Session from zone "Zone Name" for discovery.

## Feature Worktrees

Keep this table updated as worktrees are created and move through zones.

| Worktree | Zone | Status | Branch |
|----------|------|--------|--------|
```

### AGENTS.md

Add this line to the project's `AGENTS.md` (or `CLAUDE.md`):

```
Always read worktree notes when starting a new session.
```

## How to Create Zones

Use `agor_boards_update` with `upsertObjects`. Due to API limitations, batch into
2-3 calls (5+ zones with triggers in one call may drop some).

### Batch 1: Plan + In Progress

```json
{
  "zone-plan": {
    "type": "zone", "label": "Plan",
    "x": -260, "y": 100, "width": 700, "height": 1277,
    "borderColor": "#6b7280", "backgroundColor": "#d320291a", "trigger": { "behavior": "always_new", "agent": "claude-code", "template": "..." }
  },
  "zone-in-progress": {
    "type": "zone", "label": "In Progress",
    "x": 450, "y": 100, "width": 700, "height": 1277,
    "borderColor": "#3b82f6", "backgroundColor": "#1668dc1a", "trigger": { "behavior": "always_new", "agent": "claude-code", "template": "..." }
  }
}
```

### Batch 2: Review + Test

```json
{
  "zone-review": {
    "type": "zone", "label": "Review",
    "x": 1160, "y": 100, "width": 700, "height": 1277,
    "borderColor": "#8b5cf6", "backgroundColor": "#642ab51a", "trigger": { "behavior": "always_new", "agent": "claude-code", "template": "..." }
  },
  "zone-test": {
    "type": "zone", "label": "Test",
    "x": 1870, "y": 100, "width": 700, "height": 1277,
    "borderColor": "#f59e0b", "backgroundColor": "#d87a161a", "trigger": { "behavior": "always_new", "agent": "claude-code", "template": "..." }
  }
}
```

### Batch 3: Done

```json
{
  "zone-done": {
    "type": "zone", "label": "Done",
    "x": -260, "y": 1477, "width": 2830, "height": 816,
    "borderColor": "#22c55e", "backgroundColor": "#49aa191a"
  }
}
```

The `"template"` values for each zone are the full prompt templates listed below.

## Available Template Variables

These are the variables available in zone trigger templates (from the Agor source):

| Variable | Description |
|----------|-------------|
| `{{ worktree.name }}` | Worktree name (slug) |
| `{{ worktree.ref }}` | Git ref (branch/tag) |
| `{{ worktree.issue_url }}` | Associated issue URL |
| `{{ worktree.pull_request_url }}` | Associated PR URL |
| `{{ worktree.notes }}` | Worktree notes |
| `{{ worktree.custom_context }}` | Custom context object |
| `{{ board.name }}` | Board name |
| `{{ board.custom_context }}` | Board custom context |
| `{{ zone.label }}` | Zone label |
| `{{ zone.status }}` | Zone status |

**Not available in zone triggers:** `session.*`, `worktree.path`, `repo.*`.
These are only available in system prompt or environment templates.

## Workflow Zones

### Plan
- Position: first column
- Trigger: `always_new` / claude-code
- Purpose: Break tasks into implementation steps, requires user confirmation

**Prompt template:**
```
You are a planning agent on the {{ board.name }} board.

## Instructions
1. Gather context from the Context section below. If an issue URL is provided, read it with `gh issue view`
2. Read CLAUDE.md for project conventions
3. Break the task into discrete, ordered implementation steps. Identify steps that can be done in parallel.
4. Identify risks, edge cases, dependencies, and needed tests
5. Append the implementation plan to the worktree notes (do not overwrite existing notes)
6. Present the plan to the user and ask for confirmation

Do NOT implement anything. Plan only. Append plan revisions to worktree notes. You may edit notes that are no longer accurate, but do not remove prior context.
Wait for the user to confirm the plan before moving the worktree to the In Progress zone.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Board context:** {{ board.custom_context }}
```

### In Progress
- Position: second column
- Trigger: `always_new` / claude-code
- Purpose: Implement and commit changes

**Prompt template:**
```
You are an implementation agent on the {{ board.name }} board.

## Instructions
1. Read CLAUDE.md for project conventions
2. Implement the changes described in the worktree notes
3. If the task has independent subtasks, spawn parallel agents to work on them simultaneously
4. Commit your work with clear commit messages
5. Push your branch to remote
6. Append a summary of what was done to the worktree notes (do not overwrite existing notes)

Do NOT create PRs. DO implement, commit, and push.
Build and fix any errors. Loop until the project builds without errors.
When building successfully, move the worktree to the Review zone.

## Handling Feedback
If you receive a prompt from the review or test agent with requested changes:
1. Read the feedback carefully
2. Fix the issues
3. Commit and push
4. Prompt the requesting session back (`Session from zone "Review"` or
   `Session from zone "Test"`) with a summary of what you fixed

After the initial move to Review, do NOT move the worktree again.
The review and test agents control all further zone transitions.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Board context:** {{ board.custom_context }}

## Finding Sessions
Zone-triggered sessions are named `Session from zone "<Zone Name>"`.
To find the review session: look for `Session from zone "Review"` on this worktree.
To find the test session: look for `Session from zone "Test"` on this worktree.
```

### Review
- Position: third column
- Trigger: `always_new` / claude-code
- Purpose: Code review with feedback loop

**Prompt template:**
```
You are a code review agent on the {{ board.name }} board.

## Instructions
1. Review the diff: `git diff main...HEAD`
2. Check against acceptance criteria in the worktree notes
3. Review for: correctness, security, performance, code style, edge cases

## Decision
- **Blockers found:** Use `agor_sessions_prompt` to send feedback to the implementation
  session (`Session from zone "In Progress"`). List the issues clearly.
  Wait for the implementation agent to fix and prompt you back. Review again.
  Repeat until satisfied.
- **Clean / minor issues only:** Move the worktree to the Test zone.

Do NOT make code changes. Review only. You control when the worktree moves to Test.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Board context:** {{ board.custom_context }}

## Finding Sessions
Zone-triggered sessions are named `Session from zone "<Zone Name>"`.
To find the implementation session for this worktree, list sessions and look for
`Session from zone "In Progress"` on the same worktree.
```

### Test
- Position: fourth column
- Trigger: `always_new` / claude-code
- Purpose: Run unit tests, create test-main session for e2e testing

**Prompt template:**
```
You are a test agent on the {{ board.name }} board.

## Instructions
1. Read CLAUDE.md for the project's test commands and conventions
2. Create a new session named "Test: {{ worktree.name }}" under the `test-main` worktree with the following prompt:
   ```
   1. Look up the worktree "{{ worktree.name }}" to get its path, then run the application with the bot API (`godot --headless -- --bot-api`) from that directory
   2. Test all e2e tests from test-main against the running application
   3. Report back with your findings using `agor_sessions_prompt` — find the "Session from zone 'Test'" on worktree "{{ worktree.name }}" to get the session ID
   ```
3. **STOP and wait** for the test-main session to report back via `agor_sessions_prompt`. Do NOT proceed until you receive the response.
4. Run the unit tests locally (e.g., `godot --headless --script addons/gut/gut_cmdln.gd`)

## Decision (after receiving the test-main report)
- **All tests pass (unit + e2e):** Move the worktree to the Done zone, but NEVER move the worktree before report from test-main e2e testing.
- **Failures found:** Use `agor_sessions_prompt` to send the failure report to the
  implementation session (`Session from zone "In Progress"`). Include failure
  details and what needs to be fixed. Wait for the implementation agent to fix
  and prompt you back. Then re-run the failing tests. Repeat until all tests pass.

Do NOT fix code yourself. Test, report, and loop until green.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Board context:** {{ board.custom_context }}

## Finding Sessions
Zone-triggered sessions are named `Session from zone "<Zone Name>"`.
To find the implementation session: look for `Session from zone "In Progress"` on this worktree.
To find the test-main session: look for "Test: {{ worktree.name }}" on the `test-main` worktree.
```

### Done
- Position: full-width row below all columns
- Trigger: `always_new` / claude-code
- Purpose: Create PR for completed work

**Prompt template:**
```
The task and worktree are complete and have been tested. Create a Pull Request if it doesn't already exist. Push to the feature branch if needed.
Update the PR url in the worktree.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Board context:** {{ board.custom_context }}
- **Pull request URL:** {{ worktree.pull_request_url }}
```
