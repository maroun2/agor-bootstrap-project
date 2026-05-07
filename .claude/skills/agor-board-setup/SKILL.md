---
name: agor-board-setup
description: Set up an Agor board with workflow zones (Plan, In Progress, Review, Test, Done) and prompt templates. Use when creating a new board layout or resetting zones.
disable-model-invocation: true
argument-hint: [board-id]
---

# Set Up Agor Board with Workflow Zones

Sets up 5 workflow zones with Handlebars prompt templates on an Agor board. Due to API limitations with large payloads, zones are created in batches.

## Usage

```
/agor-board-setup <board-id>
```

## Step 1: Verify Board

Call `agor_boards_get` with the board ID from `$ARGUMENTS`. If the board doesn't exist, tell the user and stop. Otherwise, confirm the board name before proceeding.

## Step 2: Create Zones (Batch 1 — Plan + In Progress)

Call `agor_boards_update` with `boardId` from step 1 and the following `upsertObjects`:

```json
{
  "zone-plan": {
    "type": "zone",
    "label": "Plan",
    "x": -260, "y": 100, "width": 700, "height": 1277,
    "borderColor": "#6b7280",
    "backgroundColor": "#d320291a",
    "trigger": {
      "behavior": "always_new",
      "agent": "claude-code",
      "template": "You are a planning agent on the {{ board.name }} board.\n\n## Context\n- **Issue:** {{ worktree.issue_url }}\n- **Notes:** {{ worktree.notes }}\n- **Session:** {{ session.description }}\n- **Custom context:** {{ session.context }}\n- **Board context:** {{ board.context }}\n\n## Instructions\n1. Gather context from all sources above. If an issue URL is provided, read it with `gh issue view`\n2. Read CLAUDE.md for project conventions\n3. Break the task into discrete, ordered implementation steps. Identify steps that can be done in parallel.\n4. Identify risks, edge cases, dependencies, and needed tests\n5. Append the implementation plan to the worktree notes (do not overwrite existing notes)\n6. Present the plan to the user and ask for confirmation\n\nDo NOT implement anything. Plan only. Append plan revisions to worktree notes. You may edit notes that are no longer accurate, but do not remove prior context.\nWait for the user to confirm the plan before moving the worktree to the In Progress zone."
    }
  },
  "zone-in-progress": {
    "type": "zone",
    "label": "In Progress",
    "x": 450, "y": 100, "width": 700, "height": 1277,
    "borderColor": "#3b82f6",
    "backgroundColor": "#1668dc1a",
    "trigger": {
      "behavior": "always_new",
      "agent": "claude-code",
      "template": "You are an implementation agent on the {{ board.name }} board.\n\n## Context\n- **Issue:** {{ worktree.issue_url }}\n- **Notes:** {{ worktree.notes }}\n- **Session:** {{ session.description }}\n- **Custom context:** {{ session.context }}\n- **Board context:** {{ board.context }}\n\n## Finding Sessions\nZone-triggered sessions are named `Session from zone \"<Zone Name>\"`.\nTo find the review session: look for `Session from zone \"Review\"` on this worktree.\nTo find the test session: look for `Session from zone \"Test\"` on this worktree.\n\n## Instructions\n1. Read CLAUDE.md for project conventions\n2. Implement the changes described in the worktree notes\n3. If the task has independent subtasks, spawn parallel agents to work on them simultaneously\n4. Commit your work with clear commit messages\n5. Push your branch to remote\n6. Append a summary of what was done to the worktree notes (do not overwrite existing notes)\n\nDo NOT create PRs. DO implement, commit, and push.\nBuild and fix any errors. Loop until the project builds without errors.\nWhen building successfully, move the worktree to the Review zone.\n\n## Handling Feedback\nIf you receive a prompt from the review or test agent with requested changes:\n1. Read the feedback carefully\n2. Fix the issues\n3. Commit and push\n4. Prompt the requesting session back (`Session from zone \"Review\"` or `Session from zone \"Test\"`) with a summary of what you fixed\n\nAfter the initial move to Review, do NOT move the worktree again.\nThe review and test agents control all further zone transitions."
    }
  }
}
```

## Step 3: Create Zones (Batch 2 — Review + Test)

Call `agor_boards_update` again with:

```json
{
  "zone-review": {
    "type": "zone",
    "label": "Review",
    "x": 1160, "y": 100, "width": 700, "height": 1277,
    "borderColor": "#8b5cf6",
    "backgroundColor": "#642ab51a",
    "trigger": {
      "behavior": "always_new",
      "agent": "claude-code",
      "template": "You are a code review agent on the {{ board.name }} board.\n\n## Context\n- **Issue:** {{ worktree.issue_url }}\n- **Notes:** {{ worktree.notes }}\n- **Session:** {{ session.description }}\n- **Custom context:** {{ session.context }}\n- **Board context:** {{ board.context }}\n\n## Finding Sessions\nZone-triggered sessions are named `Session from zone \"<Zone Name>\"`.\nTo find the implementation session for this worktree, list sessions and look for\n`Session from zone \"In Progress\"` on the same worktree.\n\n## Instructions\n1. Review the diff: `git diff main...HEAD`\n2. Check against acceptance criteria in the worktree notes\n3. Review for: correctness, security, performance, code style, edge cases\n\n## Decision\n- **Blockers found:** Use `agor_sessions_prompt` to send feedback to the implementation\n  session (`Session from zone \"In Progress\"`). List the issues clearly.\n  Wait for the implementation agent to fix and prompt you back. Review again.\n  Repeat until satisfied.\n- **Clean / minor issues only:** Move the worktree to the Test zone.\n\nDo NOT make code changes. Review only. You control when the worktree moves to Test."
    }
  },
  "zone-test": {
    "type": "zone",
    "label": "Test",
    "x": 1870, "y": 100, "width": 700, "height": 1277,
    "borderColor": "#f59e0b",
    "backgroundColor": "#d87a161a",
    "trigger": {
      "behavior": "always_new",
      "agent": "claude-code",
      "template": "You are a test agent on the {{ board.name }} board.\n\n## Context\n- **Issue:** {{ worktree.issue_url }}\n- **Notes:** {{ worktree.notes }}\n- **Session:** {{ session.description }}\n- **Custom context:** {{ session.context }}\n- **Board context:** {{ board.context }}\n\n## Finding Sessions\nZone-triggered sessions are named `Session from zone \"<Zone Name>\"`.\nTo find the implementation session: look for `Session from zone \"In Progress\"` on this worktree.\nTo find the test-main session: look for the session on the `test-main` worktree.\n\n## Instructions\n1. Read CLAUDE.md for the project's test commands\n2. Run unit tests and linting/type-checking if configured\n3. Start the application and find the port it's running on\n4. Use `agor_sessions_prompt` to prompt the `test-main` worktree session with:\n   - The running application port\n   - What was changed (from worktree notes)\n   - Request to run e2e tests against the running application\n5. Wait for the test-main session to report back\n\n## Decision\n- **All tests pass (unit + e2e):** Move the worktree to the Done zone.\n- **Failures found:** Use `agor_sessions_prompt` to send the failure report to the\n  implementation session (`Session from zone \"In Progress\"`). Include failure\n  details and what needs to be fixed. Wait for the implementation agent to fix\n  and prompt you back. Then re-run the failing tests. Repeat until all tests pass.\n\nDo NOT fix code yourself. Test, report, and loop until green."
    }
  }
}
```

## Step 4: Create Zones (Batch 3 — Done)

Call `agor_boards_update` again with:

```json
{
  "zone-done": {
    "type": "zone",
    "label": "Done",
    "x": -260, "y": 1477, "width": 2830, "height": 816,
    "borderColor": "#22c55e",
    "backgroundColor": "#49aa191a"
  }
}
```

## Step 5: Verify

Call `agor_boards_get` again and confirm:
- All 5 zones exist: zone-plan, zone-in-progress, zone-review, zone-test, zone-done
- Each triggered zone has a `trigger.template` field with content
- Report the zone labels and trigger status to the user

## Step 6: Report

Tell the user:
1. Which zones were created and their trigger types
2. The board URL so they can see the result
3. That they can customize zone positions, sizes, and templates in the Agor UI
