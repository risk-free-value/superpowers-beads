---
name: beads-checkpoint
description: Use at 70%+ token usage, before context switch, or when user steps away - preserves work state in beads with structured notes for compaction survival
---

# Beads Checkpoint

## Overview

Write structured notes to beads **BEFORE** compaction happens. Notes written after context loss are low quality.

**Key insight:** Precompact hook is too late. By then your context is already degrading. Checkpoint proactively at 70%.

## Checkpoint Triggers

| Trigger | Action |
|---------|--------|
| 70% token usage | Proactive checkpoint (best quality) |
| 85% token usage | Warning - checkpoint immediately |
| 90% token usage | Emergency checkpoint |
| User stepping away | Checkpoint before they leave |
| Major milestone | Update notes with progress |
| Hit a blocker | Capture what was tried |
| Task transition | Update before switching tasks |

## Protocol

### 1. List In-Progress Issues

```bash
bd list --status in_progress --json
```

### 2. Write Structured Notes

For EACH in-progress issue:

```bash
bd update <id> --notes "COMPLETED: [specific deliverables - files, functions, tests]
IN PROGRESS: [current state + immediate next step]
NEXT: [concrete action with file paths if known]
BLOCKERS: [what's preventing progress, if any]
KEY DECISIONS: [important context with rationale]"
```

**Example:**
```bash
bd update Superpowers-xyz --notes "COMPLETED: Created skills/plan-to-beads/SKILL.md with TDD methodology. Added test-baseline.md.
IN PROGRESS: Testing skill with subagent to verify compliance.
NEXT: Run verification test, then close bead.
BLOCKERS: None.
KEY DECISIONS: Skill replaces writing-plans for beads workflows (not alongside it)."
```

### 3. Quality Check

Ask yourself: **"If a fresh agent started with ONLY these notes, could they continue effectively?"**

If no → add more context. Be specific about:
- What files were created/modified
- What decisions were made and WHY
- What the concrete next step is

### 4. Commit Beads State

```bash
git add .beads/
git commit -m "chore: checkpoint beads state"
```

(Ask user before committing per CLAUDE.md)

## Quick Reference

| Field | What to Write | Bad Example | Good Example |
|-------|---------------|-------------|--------------|
| COMPLETED | Specific deliverables | "Made progress" | "Created auth middleware at src/middleware/auth.ts" |
| IN PROGRESS | Current state + next step | "Working on it" | "Middleware passes tests, adding error handling" |
| NEXT | Concrete action | "Continue" | "Add 401 response for expired tokens" |
| BLOCKERS | What's stuck | "Some issues" | "Need JWT_SECRET env var, asked user" |
| KEY DECISIONS | Context + rationale | "Decided stuff" | "Using RS256 for JWT (team standard)" |

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'll remember where we were" | You won't exist next session. Notes are for the next agent. |
| "The summary will capture it" | Summaries lose detail. Structured notes preserve it. |
| "User seems in a hurry" | 30 seconds to checkpoint saves 30 minutes of confusion. |
| "It's just notes, no need to commit" | Uncommitted notes can be lost. Commit is cheap. |
| "70% is too early" | 70% is ideal. 90% is too late. Context degrades gradually. |
| "I'll checkpoint at the end" | You might not reach "the end". Checkpoint at milestones. |

## Reference

For bd commands: user's `~/.claude/skills/beads/`.
