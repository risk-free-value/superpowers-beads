---
name: executing-plans
description: Use when beads contain implementation tasks ready for execution - loads tasks from bd, executes in batches with review checkpoints, updates bd status throughout, checkpoints at 70% tokens
---

# Executing Plans

## Overview

Load tasks from beads, execute in batches, report for review between batches.

**Core principle:** Batch execution with checkpoints. Beads track persistent state, TodoWrite tracks session visibility.

**Announce at start:** "I'm using the executing-plans skill to implement these tasks."

## The Process

### Step 1: Load Tasks from Beads

1. Find ready tasks:
   ```bash
   bd ready --json
   ```

2. Read each task's full details:
   ```bash
   bd show <task-id>
   ```

3. Review tasks critically - identify concerns about:
   - Missing information in WHAT/HOW/ACCEPTANCE
   - Unclear dependencies
   - Gaps in acceptance criteria

4. If concerns: Raise with your human partner before starting

5. Create TodoWrite mirroring first batch (default: 3 tasks)

### Step 2: Execute Task

For each task:

1. **Claim in beads:**
   ```bash
   bd update <task-id> --status in_progress
   ```

2. **Mark TodoWrite item in_progress**

3. **Follow implementation** from task description:
   - WHAT: The goal
   - HOW: Implementation steps
   - CONTEXT: Background needed
   - ACCEPTANCE: Verification criteria

4. **Run verifications** as specified in ACCEPTANCE

5. **Update notes at milestones:**
   ```bash
   bd update <task-id> --notes "COMPLETED: X. IN PROGRESS: Y. NEXT: Z."
   ```

6. **Mark TodoWrite complete**

### Step 3: Complete Task

When task done:

1. **Close in beads:**
   ```bash
   bd close <task-id> --reason "Implemented per acceptance criteria"
   ```

2. **Ask before committing:**
   ```
   Ready to commit? Changes:
   - [list files]

   Commit message: [proposed message]

   [Proceed / Edit / Skip]
   ```

3. **If approved:** Commit code + .beads/ together

### Step 4: Report and Continue

When batch complete:
- Show what was implemented
- Show verification output
- Say: "Ready for next batch?"

Based on feedback:
- Apply changes if needed
- Load next batch into TodoWrite
- Repeat until all tasks complete

### Step 5: Complete Development

After all tasks complete and verified:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## Checkpoint Integration

**At 70% token usage or after each batch:**

1. Update bd notes on all in_progress tasks:
   ```bash
   bd update <task-id> --notes "COMPLETED: X. IN PROGRESS: Y. NEXT: Z. BLOCKERS: W."
   ```

2. Commit .beads/ with checkpoint message

3. **REQUIRED SUB-SKILL:** Use superpowers:beads-checkpoint for full protocol

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker mid-task (missing dependency, test fails, instruction unclear)
- Task has critical gaps in WHAT/HOW/ACCEPTANCE
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.**

## When to Revisit Earlier Steps

**Return to Load (Step 1) when:**
- Partner updates task descriptions based on your feedback
- New tasks added to beads
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Remember

- Load from beads, not markdown files
- Claim task (in_progress) before starting
- Close task when complete
- Update notes at milestones and checkpoints
- TodoWrite mirrors beads for session visibility
- Between batches: just report and wait
- Stop when blocked, don't guess
- **Ask before committing** - show changes and proposed message first
- Always commit .beads/ with code changes
