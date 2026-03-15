# Bootstrap - Project Coordinator

Instructions for the coordinator agent after it fetches the docs.

## Before Starting

Check which steps have already been completed. Look for these artifacts:
- [ ] Board zones exist (Plan, In Progress, Review, Test, Done)
- [ ] Coordinator worktree exists on the board
- [ ] `roadmap.md` exists in the repo
- [ ] `CLAUDE.md` references `roadmap.md`
- [ ] `api.md` exists in the repo
- [ ] `CLAUDE.md` references `api.md`
- [ ] Designer agent asked about / set up (optional)
- [ ] Test agent worktree exists and is placed outside zones
- [ ] Phase 1 worktrees are created and placed in Plan zone

Skip any step that is already complete. Pick up from the first unchecked item.

## Process

Follow these steps in order. Do not skip ahead. Each step requires
user confirmation before proceeding to the next.

### Step 0: Board Setup
Read `doc/board-setup.md` and set up the board:
- [ ] Create workflow zones (Plan, In Progress, Review, Test, Done)
- [ ] Check if a `coordinator` worktree already exists on the board
- [ ] If it does NOT exist: create it, place it outside/above the zones
- [ ] If it already exists: skip creation, continue with the existing one
- [ ] Set the coordinator worktree notes (see "Coordinator Worktree Notes" in `doc/board-setup.md`)
- [ ] Add `Always read worktree notes when starting a new session.` to the project's `AGENTS.md` or `CLAUDE.md`

### Step 1: Roadmap
Read `doc/create-roadmap.md` and follow it completely:
- [ ] Market research
- [ ] Interactive concept refinement with the user
- [ ] Draft and finalize the roadmap
- [ ] Save `roadmap.md`, update `CLAUDE.md`, commit and merge to main

### Step 2: API Documentation
Read `doc/create-api-doc.md` and follow it completely:
- [ ] Draft API documentation based on the roadmap
- [ ] Interactive review with the user
- [ ] Save `api.md`, update `CLAUDE.md`, commit and merge to main

### Step 3: Designer Agent (Optional)
Read `doc/create-designer.md` and follow it completely:
- [ ] Ask the user if the project needs graphic asset generation
- [ ] If yes: set up image generation service, create designer worktree
- [ ] If no: skip and continue

### Step 4: Test Agent
Read `doc/create-tests.md` and follow it completely:
- [ ] Create test agent worktree
- [ ] Run it through Plan -> In Progress -> Review
- [ ] Place it outside zones as permanent test agent

### Step 5: Phase 1 Tasks
Read `doc/create-phase1.md` and follow it completely:
- [ ] Create worktrees for all Phase 1 tasks from the roadmap
- [ ] Set detailed notes on each with acceptance criteria
- [ ] Place them in the Plan zone

## Important
- Every step is interactive — discuss with the user before finalizing
- Do NOT rush through steps
- Each document saved to the repo must be committed and merged to main
- Reference all key documents in `CLAUDE.md` so all agents can find them
