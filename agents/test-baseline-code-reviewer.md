# Code Reviewer Beads Integration - Test Baseline

## Current Behavior

The agent currently:
1. Compares implementation against planning document
2. Reviews code quality, architecture, documentation
3. Categorizes issues as Critical/Important/Suggestions
4. Communicates deviations and recommendations

## Problem

Without beads integration:
- Reviews against vague "plan" references
- No explicit acceptance criteria checking
- No recommendation on whether to close bd task
- No discovery of follow-up work as new beads

## Required Changes

### 7. Beads Acceptance Check (new section)

If bd task ID provided:
1. Run `bd show <task-id>`
2. Extract ACCEPTANCE criteria
3. Verify each criterion is met
4. Track X/Y criteria met

### 8. Beads Status Output (new section)

Add to review output:
- Task ID and title
- Acceptance criteria table (criterion, status, notes)
- Recommendation: close/continue/follow-up

### Communication Protocol Updates

- If bd task ID provided, always include Beads Status
- Recommend closing if all acceptance met
- Recommend follow-up beads for discovered work

## Acceptance Criteria

- [ ] Added Beads Acceptance Check section
- [ ] Added Beads Status to review output format
- [ ] Checks each acceptance criterion
- [ ] Recommends close/continue/follow-up based on criteria
- [ ] Handles case where no bd task ID provided (skip beads check)
