# Executing Plans Beads Integration - Test Baseline

## Current Behavior (Pre-Beads)

The skill currently:
1. Reads from markdown plan files
2. Uses TodoWrite for tracking (ephemeral, session-only)
3. No persistent state across sessions
4. Batch execution with human review checkpoints

## Problem

Without beads integration:
- Work state lost on context compaction
- No way to resume interrupted work
- No dependency tracking between tasks
- No persistent acceptance criteria verification

## Required Changes

### Step 1: Load from Beads (not markdown)
- `bd ready --json` to find unblocked tasks
- `bd show <task-id>` to read full task details
- Task has WHAT/WHY/HOW/CONTEXT/ACCEPTANCE structure

### Step 2: Execute with bd Status Updates
- `bd update <id> --status in_progress` when starting
- Follow WHAT/HOW from task description
- Verify against ACCEPTANCE criteria
- `bd update <id> --notes "..."` at milestones

### Step 3: Complete Task
- `bd close <id> --reason "..."` when done
- Ask before committing (already in skill)
- Commit includes .beads/ directory

### Checkpoint Integration
- At 70% token usage: checkpoint all in_progress tasks
- Use beads-checkpoint protocol
- Structured notes: COMPLETED/IN PROGRESS/NEXT/BLOCKERS

### TodoWrite Role
- Still used for session visibility
- Mirrors bd task status
- Ephemeral complement to persistent beads

## Acceptance Criteria

- [ ] Loads tasks from bd ready/bd show (not markdown)
- [ ] Marks bd task in_progress when starting
- [ ] Closes bd task when complete
- [ ] Updates bd notes at checkpoints
- [ ] Asks before committing
- [ ] Maintains TodoWrite for session progress
- [ ] References beads skill for bd commands
