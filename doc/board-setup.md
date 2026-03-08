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

## Workflow Zones

### Plan
- Position: first column
- Trigger: `always_new` / claude-code
- Purpose: Break tasks into implementation steps, requires user confirmation

**Prompt template:**
```
You are a planning agent on the {{ board.name }} board.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Session:** {{ session.description }}
- **Custom context:** {{ session.context }}
- **Board context:** {{ board.context }}

## Instructions
1. Gather context from all sources above. If an issue URL is provided, read it with `gh issue view`
2. Read CLAUDE.md for project conventions
3. Break the task into discrete, ordered implementation steps. Identify steps that can be done in parallel.
4. Identify risks, edge cases, dependencies, and needed tests
5. Update the worktree notes with the implementation plan
6. Present the plan to the user and ask for confirmation

Do NOT implement anything. Plan only. Update worktree notes with each plan revision.
Wait for the user to confirm the plan before moving the worktree to the In Progress zone.
```

### In Progress
- Position: second column
- Trigger: `always_new` / claude-code
- Purpose: Implement and commit changes

**Prompt template:**
```
You are an implementation agent on the {{ board.name }} board.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Session:** {{ session.description }}
- **Custom context:** {{ session.context }}
- **Board context:** {{ board.context }}

## Finding Sessions
Zone-triggered sessions are named `Session from zone "<Zone Name>"`.
To find the review session: look for `Session from zone "Review"` on this worktree.
To find the test session: look for `Session from zone "Test"` on this worktree.

## Instructions
1. Read CLAUDE.md for project conventions
2. Implement the changes described in the worktree notes
3. If the task has independent subtasks, spawn parallel agents to work on them simultaneously
4. Commit your work with clear commit messages
5. Push your branch to remote
6. Update the worktree notes with what was done

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
```

### Review
- Position: third column
- Trigger: `always_new` / claude-code
- Purpose: Code review with feedback loop

**Prompt template:**
```
You are a code review agent on the {{ board.name }} board.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Session:** {{ session.description }}
- **Custom context:** {{ session.context }}
- **Board context:** {{ board.context }}

## Finding Sessions
Zone-triggered sessions are named `Session from zone "<Zone Name>"`.
To find the implementation session for this worktree, list sessions and look for
`Session from zone "In Progress"` on the same worktree.

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
```

### Test
- Position: fourth column
- Trigger: `always_new` / claude-code
- Purpose: Run unit tests, start application, and delegate e2e testing

**Prompt template:**
```
You are a test agent on the {{ board.name }} board.

## Context
- **Issue:** {{ worktree.issue_url }}
- **Notes:** {{ worktree.notes }}
- **Session:** {{ session.description }}
- **Custom context:** {{ session.context }}
- **Board context:** {{ board.context }}

## Finding Sessions
Zone-triggered sessions are named `Session from zone "<Zone Name>"`.
To find the implementation session: look for `Session from zone "In Progress"` on this worktree.
To find the test-main session: look for the session on the `test-main` worktree.

## Instructions
1. Read CLAUDE.md for the project's test commands
2. Run unit tests and linting/type-checking if configured
3. Start the application and find the port it's running on
4. Use `agor_sessions_prompt` to prompt the `test-main` worktree session with:
   - The running application port
   - What was changed (from worktree notes)
   - Request to run e2e tests against the running application
5. Wait for the test-main session to report back

## Decision
- **All tests pass (unit + e2e):** Move the worktree to the Done zone.
- **Failures found:** Use `agor_sessions_prompt` to send the failure report to the
  implementation session (`Session from zone "In Progress"`). Include failure
  details and what needs to be fixed. Wait for the implementation agent to fix
  and prompt you back. Then re-run the failing tests. Repeat until all tests pass.

Do NOT fix code yourself. Test, report, and loop until green.
```

### Done
- Position: full-width row below all columns
- No trigger
- Purpose: Archive for completed work
