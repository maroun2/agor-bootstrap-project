# Create Phase 1 Task Worktrees

After the test agent is set up, the coordinator creates worktrees for all
Phase 1 tasks from the roadmap.

## Step 1: Read the Roadmap

Read `roadmap.md` and extract all Phase 1 tasks from the work breakdown section.

## Step 2: Create Worktrees

For each Phase 1 task:
1. Create a worktree with a descriptive name (e.g., `feature-root-network`, `feature-seasonal-cycle`)
2. Set detailed notes including:
   - Task description from the roadmap
   - Relevant acceptance criteria
   - API references from `api.md` (if applicable)
   - Dependencies on other tasks (if any)
3. Place the worktree in the **Plan zone**

## Step 3: Parallelism

Identify which tasks can run in parallel (no dependencies on each other)
and note this in each worktree's notes so the planning agents can coordinate.

## Step 4: Report

Tell the user:
- How many worktrees were created
- Which tasks are independent (can run in parallel)
- Which have dependencies (must be sequenced)
- Suggested order of execution

## After This

The coordinator's bootstrap is complete. From here:
- Worktrees flow through Plan → In Progress → Review → Test → Done
- The coordinator monitors progress and can create additional worktrees as needed
- When Phase 1 is complete, the coordinator starts Phase 2 from the roadmap
