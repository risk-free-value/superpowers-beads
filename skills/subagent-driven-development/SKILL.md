---
name: subagent-driven-development
description: Use when executing beads tasks with independent work in the current session - dispatches fresh subagent per task with bd acceptance criteria, code review between tasks, updates bd status throughout
---

# Subagent-Driven Development

Execute beads tasks by dispatching fresh subagent per task, with code review after each.

**Core principle:** Fresh subagent per task + review between tasks = high quality, fast iteration

## Overview

**vs. Executing Plans (parallel session):**
- Same session (no context switch)
- Fresh subagent per task (no context pollution)
- Code review after each task (catch issues early)
- Faster iteration (no human-in-loop between tasks)

**When to use:**
- Staying in this session
- Tasks are mostly independent
- Want continuous progress with quality gates

**When NOT to use:**
- Need to review tasks first (use executing-plans)
- Tasks are tightly coupled (manual execution better)
- Tasks need revision (brainstorm first)

## The Process

### 1. Load Tasks from Beads

1. Find ready tasks:
   ```bash
   bd ready --json
   ```

2. Read each task's full details:
   ```bash
   bd show <task-id>
   ```

3. Create TodoWrite mirroring tasks for session visibility

### 2. Execute Task with Subagent

For each task:

**First, claim in beads:**
```bash
bd update <task-id> --status in_progress
```

**Read full task details:**
```bash
bd show <task-id>  # Get WHAT/WHY/HOW/CONTEXT/ACCEPTANCE
```

**Dispatch fresh subagent with full context:**
```
Task tool (general-purpose):
  description: "Implement: [task title]"
  prompt: |
    You are implementing a task from beads.

    Task: [task title]

    WHAT: [paste WHAT section]

    WHY: [paste WHY section]

    HOW: [paste HOW section]

    CONTEXT: [paste CONTEXT section]

    ACCEPTANCE CRITERIA:
    [paste ACCEPTANCE section - these are your success criteria]

    Your job:
    1. Implement exactly what the task specifies
    2. Meet ALL acceptance criteria
    3. Write tests (following TDD)
    4. Verify implementation works
    5. DO NOT commit - report back for review first

    Work from: [directory]

    Report: What you implemented, acceptance criteria status, test results, files changed, any issues
```

**After subagent reports:**

If successful:
1. Ask user to commit:
   ```
   Ready to commit? Changes:
   - [files changed from subagent report]

   Commit message: [proposed message]

   [Proceed / Edit / Skip]
   ```
2. After commit, close bd task:
   ```bash
   bd close <task-id> --reason "[summary: what was implemented, acceptance met]"
   ```

If issues:
1. Update bd notes with what was tried:
   ```bash
   bd update <task-id> --notes "ATTEMPTED: [what subagent tried]. FAILED: [what went wrong]. NEXT: [suggested fix]."
   ```
2. Dispatch fix subagent OR escalate to user

### 3. Review Subagent's Work

**Dispatch code-reviewer subagent:**
```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from subagent's report]
  PLAN_OR_REQUIREMENTS: bd task <task-id> - include acceptance criteria
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
  BD_TASK_ID: <task-id>  # Reviewer can run bd show to check acceptance
```

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment

**Reviewer checks:**
- All acceptance criteria from bd task met
- Implementation matches WHAT/HOW
- Tests cover acceptance criteria

### 4. Apply Review Feedback

**If issues found:**
- Fix Critical issues immediately
- Fix Important issues before next task
- Note Minor issues

**Dispatch follow-up subagent if needed:**
```
"Fix issues from code review: [list issues]"
```

### 5. Mark Complete, Next Task

- Mark task as completed in TodoWrite
- Move to next task
- Repeat steps 2-5

### 6. Final Review

After all tasks complete, dispatch final code-reviewer:
- Reviews entire implementation
- Checks all bd acceptance criteria met across tasks
- Validates overall architecture

### 7. Complete Development

After final review passes:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## Checkpoint Integration

**At 70% token usage:**
1. Update bd notes on all in_progress tasks
2. Commit .beads/
3. **REQUIRED SUB-SKILL:** Use superpowers:beads-checkpoint

## Red Flags

**Never:**
- Skip code review between tasks
- Proceed with unfixed Critical issues
- Dispatch multiple implementation subagents in parallel (conflicts)
- Implement without reading bd task first
- Close bd task without meeting acceptance criteria

**If subagent fails task:**
- Update bd notes with what was tried
- Dispatch fix subagent with specific instructions
- Don't try to fix manually (context pollution)

## Integration

**Required workflow skills:**
- **plan-to-beads** - Creates the beads tasks this skill executes
- **requesting-code-review** - Review after each task (see Step 3)
- **finishing-a-development-branch** - Complete development after all tasks (see Step 7)
- **beads-checkpoint** - Checkpoint at 70% tokens

**Subagents must use:**
- **test-driven-development** - Subagents follow TDD for each task

**Alternative workflow:**
- **executing-plans** - Use for parallel session instead of same-session execution

See code-reviewer template: requesting-code-review/code-reviewer.md
