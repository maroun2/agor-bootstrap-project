# Create Test Agent

After API documentation is finalized and merged, the coordinator sets up the
main test agent. This agent will live permanently outside the zones and test
other worktrees as they complete their work.

Note: At this point no project code exists yet. The test agent creates
end-to-end tests against the API documentation — not unit tests.

## Step 1: Create Test Worktree

Create a worktree for the main test agent:
- Name: `test-main` (or similar)
- Set notes with:
  - Testing strategy from `roadmap.md`
  - API spec reference from `api.md`
  - Instructions: create e2e test framework that validates against `api.md`
  - No unit tests needed at this point — those come with feature implementation
  - Tests should be runnable against any feature branch to verify API compliance

## Step 2: Run Through Pipeline

Move the test worktree through the normal workflow:
- **Plan zone** — planning agent designs the e2e test structure based on `api.md`
- **In Progress zone** — implementation agent builds the test framework and API contract tests
- **Review zone** — review agent checks test coverage against `api.md`

## Step 3: Place Outside Zones

Once the test agent passes review:
- Move the worktree outside the workflow zones (persistent position on the board)
- This becomes the main test agent for the project
- Other worktrees will be tested against this agent's test suite

## Test Agent Responsibilities

When a feature worktree needs testing:
- Run e2e tests against the feature branch
- Validate API compliance against `api.md`
- Check acceptance criteria from the worktree notes (each task worktree includes its own AC)
- Cross-reference with `roadmap.md` for overall project acceptance criteria
- Report: PASS/FAIL, summary counts, failure details
- Do NOT fix failures — report only

Then proceed to `doc/create-phase1.md`.
