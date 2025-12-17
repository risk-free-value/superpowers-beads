# Finishing a Development Branch Beads Integration - Test Baseline

## Current Behavior

The skill currently:
1. Verifies tests pass
2. Determines base branch
3. Presents 4 options (merge, PR, keep, discard)
4. Executes chosen option
5. Cleans up worktree

## Problem

Without beads integration:
- bd tasks left in_progress after merge
- Epic not closed
- .beads/ not committed with code
- Beads state lost on merge

## Required Changes

### Beads Completion Section (before Step 3)

1. Close all in_progress tasks
2. Close the epic
3. Stage .beads/ for commit

### Modify Commit Behavior

- Include .beads/ in merge commit (Option 1)
- Include .beads/ in PR commit (Option 2)
- Ask before committing

## Acceptance Criteria

- [ ] Beads Completion section added
- [ ] Closes all in_progress tasks
- [ ] Closes epic
- [ ] .beads/ included in commit
- [ ] Asks before committing
