---
name: plan-to-beads
description: Use after brainstorming when design is approved and you need implementation tasks - converts design document into granular, self-documenting beads with full dependency structure instead of markdown plans
---

# Plan to Beads

## Overview

Convert approved design into **beads**, not markdown implementation plans. Each bead must be self-contained - a fresh agent with ONLY that bead should understand what to do and why.

**This skill REPLACES writing-plans** for beads-integrated workflows. Beads ARE the implementation plan.

## When to Use

- Design document approved (brainstorming complete)
- User says "let's implement" or "turn this into a plan"
- Ready to decompose design into executable tasks

**NOT writing-plans.** If you catch yourself creating `docs/plans/...-implementation.md`, STOP. Use this skill instead.

## Process

### 1. Read Design Document

Understand: goals, architecture decisions, components, constraints.

### 2. Create Epic

```bash
bd create "Feature: [name]" -t epic -p [priority] --json
bd update <epic-id> --description "Goal: [what and why]
Architecture: [key decisions]
Design doc: docs/plans/YYYY-MM-DD-<topic>-design.md
Success criteria: [what done looks like]"
```

### 3. Decompose into Granular Tasks

For EACH logical unit of work:

```bash
bd create "[Specific task]" -t task --json
bd dep add <epic-id> <task-id> --type parent-child
```

### 4. Task Description Format (CRITICAL)

Each task MUST include this structure:

```
WHAT: Specific deliverable (one action, not "implement auth")
WHY: How it serves the goal
HOW: Implementation approach with file paths
CONTEXT: Background, constraints, relevant code patterns
ACCEPTANCE:
- [ ] Criterion 1
- [ ] Criterion 2
```

**Example:**
```bash
bd update <task-id> --description "WHAT: Create user session validation middleware
WHY: Required for protected routes per design section 3.2
HOW: Add middleware at src/middleware/auth.ts using existing jwt.verify pattern
CONTEXT: JWT secret from env.AUTH_SECRET, pattern in src/middleware/logging.ts
ACCEPTANCE:
- [ ] Middleware validates JWT from Authorization header
- [ ] Returns 401 for missing/invalid tokens
- [ ] Passes decoded user to req.user
- [ ] Unit tests cover valid, expired, missing token cases"
```

### 5. Add Blocking Dependencies

When Task B requires Task A:

```bash
bd dep add <task-a-id> <task-b-id> --type blocks
```

### 6. Verify Structure

```bash
bv --robot-plan  # Shows logical execution order
bv --robot-insights  # Check for cycles
```

### 7. Quality Check

For EACH bead, ask: **"Could a fresh agent with ONLY this bead implement it correctly?"**

If no -> add more context to description.

## Quick Reference

| Step | Command | Purpose |
|------|---------|---------|
| Epic | `bd create "Feature: X" -t epic` | Container for feature |
| Task | `bd create "Do Y" -t task` | Single unit of work |
| Parent-child | `bd dep add <epic> <task> --type parent-child` | Task belongs to epic |
| Blocking | `bd dep add <a> <b> --type blocks` | B waits for A |
| Verify | `bv --robot-plan` | Check execution order |

## What NOT to Create

- Markdown implementation plans (use beads)
- Vague tasks ("implement auth" - too broad)
- Tasks missing WHY or CONTEXT
- Tasks without ACCEPTANCE criteria
- Duplicate content from design doc (reference it instead)

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using writing-plans | STOP. This skill replaces it. Beads ARE the plan. |
| Vague task titles | Be specific: "Create JWT middleware" not "Add auth" |
| Missing CONTEXT | Include file paths, patterns to follow, constraints |
| No dependencies | Add parent-child for epic, blocks for sequencing |
| Skipping verification | Always run bv --robot-plan before done |

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Feature is small, markdown is fine" | Beads survive compaction. Markdown doesn't. Size irrelevant. |
| "I know the codebase, CONTEXT is overkill" | You won't be here next session. Next agent needs CONTEXT. |
| "I'll add ACCEPTANCE later" | No. ACCEPTANCE criteria ARE the definition of done. Add now. |
| "writing-plans has more detail" | Beads with proper descriptions have MORE context, survive compaction. |
| "Dependencies are obvious" | Make them explicit. bv --robot-plan needs them. Obvious to you ≠ obvious to system. |
| "This is faster without all the structure" | Shortcuts now = confusion later. 2 minutes per bead saves hours. |

## Reference

For bd/bv commands: user's `~/.claude/skills/beads/` and `~/.claude/skills/using-bv/`.
