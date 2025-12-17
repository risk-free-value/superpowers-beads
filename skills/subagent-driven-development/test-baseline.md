# Subagent-Driven Development Beads Integration - Test Baseline

## Current Behavior

The skill currently:
1. Loads plan from markdown file (line 34)
2. Subagent reads task from plan file (line 45)
3. No bd status updates
4. Code reviewer checks against plan file (line 78)
5. References writing-plans as the plan source (line 198)

## Problem

Without beads integration:
- Subagent must read plan file (extra context, potential staleness)
- No persistent tracking of task completion
- No acceptance criteria contract
- Code reviewer can't check against bd acceptance criteria

## Required Changes

### 1. Load Tasks from Beads
- `bd ready --json` to find tasks
- `bd show <task-id>` to get full details

### 2. Updated Subagent Prompt
- Pass full bd task description (WHAT/WHY/HOW/CONTEXT)
- Pass ACCEPTANCE criteria explicitly
- Subagent verifies against acceptance

### 3. After Subagent Success
- `bd close <task-id> --reason "..."`
- Ask before committing

### 4. After Subagent Failure
- `bd update <task-id> --notes "ATTEMPTED: ... FAILED: ... NEXT: ..."`
- Dispatch fix subagent or escalate

### 5. Code Reviewer Integration
- Pass bd task ID so reviewer can check acceptance criteria
- Reviewer verifies all acceptance criteria met

## Acceptance Criteria

- [ ] Reads bd task before dispatching subagent
- [ ] Passes full task description and acceptance criteria to subagent
- [ ] Closes bd task after successful completion
- [ ] Updates bd notes if subagent fails
- [ ] Asks before committing
- [ ] Passes bd task ID to code-reviewer
