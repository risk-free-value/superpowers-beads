---
name: session-start-with-beads
description: Use at session start when project has .beads/ directory - checks persistent work state BEFORE skill selection to continue in-progress work or select from ready tasks
---

# Session Start with Beads

## Overview

Check beads state BEFORE skill selection. Beads determines WHAT to work on, skills determine HOW.

Work persists across sessions. Previous conversations may have left work in-progress or ready.

## Protocol

### Step 1: Check State (FIRST, before anything else)

```bash
bd ready --json
bd list --status in_progress --json
```

Run both in parallel. Do this BEFORE checking skills.

### Step 2a: If In-Progress Work Exists

For each in-progress issue:
1. Run `bd show <id>`
2. Read the **notes** field (structured: COMPLETED/IN PROGRESS/NEXT/BLOCKERS)
3. Report: "Continuing **[title]**. Last: [COMPLETED]. Next: [NEXT]."
4. Ask: "Continue, or work on something else?"

### Step 2b: If Nothing In-Progress But Ready Work Exists

1. Run `bv --robot-plan` for prioritization (if available)
2. Present top 3 ready tasks with priorities
3. Ask: "Which to work on? Or something else?"

### Step 3: Claim Work

When user confirms: `bd update <id> --status in_progress`

### Step 4: THEN Check Skills

After establishing WHAT to work on, check skills for HOW.

## Key Principle

```
Beads = WHAT (persistent across sessions)
Skills = HOW (workflow for current task)
```

Check beads first. Always.

## When To Skip

Skip this protocol ONLY if:
- Project has no `.beads/` directory
- User explicitly says "ignore beads" or "start fresh"

**NOT valid reasons to skip:**
- "User asked a specific question" → Check beads anyway, question may relate to tracked work
- "I'll check beads after" → No. Beads first determines context
- "Seems urgent" → 5 seconds to check bd won't hurt

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Check skills before beads | Beads first, always |
| Ask "what to work on?" without checking | Run bd commands first |
| Skip reading notes field | Notes contain critical context |
| Assume fresh start | Check for in-progress work |

## Reference

For detailed bd/bv commands: user's `~/.claude/skills/beads/` and `~/.claude/skills/using-bv/` skills.
