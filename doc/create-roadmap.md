# Create Roadmap

The coordinator follows these steps to create the project roadmap.

## Step 1: Understand the Project

Read the worktree notes for the project concept, name, and any existing context.

## Step 2: Market Research

Search the web for similar projects in this space:
- What makes the top-rated ones successful?
- What do users love? What do they complain about?
- What's missing in the market that this project could fill?
- Present findings to the user.

## Step 3: Concept Refinement (INTERACTIVE)

This is the most important step. Work with the user iteratively:
- Present your research findings and initial concept assessment
- Ask the user about their vision — what excites them most?
- Discuss core mechanics/features one by one
- Propose ideas, get feedback, adjust
- Challenge assumptions — suggest alternatives the user may not have considered
- Discuss target audience and what would make them love this
- Design direction / visual style (if applicable)
- Keep going until the user says they're happy with the concept

Do NOT rush this. Multiple rounds of discussion are expected.

## Step 4: Draft the Roadmap

Once the concept is agreed upon, plan the full project:

**Tech Stack**
- Language, framework, engine recommendation
- Architecture overview
- Key technical risks

**MVP Scope**
- Minimum features for a playable/usable first version
- What to cut from v1
- Definition of "done" for MVP

**Feature Roadmap**
- Phase 1: MVP
- Phase 2: Core expansion
- Phase 3: Polish & launch
- Phase 4: Post-launch

**Testing Strategy**
- How will the project be tested? (unit, integration, e2e)
- How does the test agent get feedback from the whole project running?
- What frameworks/tools for testing?
- Acceptance criteria format for each feature

**Work Breakdown**
- Break work into discrete worktrees/tasks
- Identify what can be done in parallel
- Suggest order of implementation

## Step 5: Finalize Roadmap

Once the user accepts the roadmap:
1. Save the full roadmap to `roadmap.md` in the repo root
2. Add a reference to `roadmap.md` in `CLAUDE.md` (create if needed) so all agents know about it
3. Commit both files with a clear message
4. Merge the branch to main

Then proceed to `doc/create-api-doc.md`.
