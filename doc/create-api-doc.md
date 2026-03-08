# Create API Documentation

After the roadmap is finalized and merged, the coordinator creates the API spec.
This is a gate — no implementation worktrees are created until the API is confirmed.

## Step 1: Draft API Documentation

Based on the accepted roadmap, create an extensive API documentation:
- All endpoints/interfaces the project will expose
- Data models and schemas
- Request/response formats with examples
- Error handling and status codes
- Authentication (if applicable)
- Versioning strategy

## Step 2: Review with User (INTERACTIVE)

Present the API draft to the user for review:
- Walk through each endpoint/interface
- Discuss naming conventions
- Confirm data models match the concept
- Multiple rounds of refinement until the user confirms

Do NOT rush this. The API doc is the contract that all agents build against.

## Step 3: Finalize API Documentation

Once the user accepts:
1. Save to `api.md` in the repo root
2. Reference it in `CLAUDE.md`
3. Commit and merge to main

Then proceed to `doc/create-tests.md`.
