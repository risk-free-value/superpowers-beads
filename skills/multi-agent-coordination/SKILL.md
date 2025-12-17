---
name: multi-agent-coordination
description: Use when multiple agents need to work concurrently on different tasks - coordinates via beads atomic task claims instead of git worktrees, keeping state unified
---

# Multi-Agent Coordination

## Overview

Agents coordinate via **beads status** instead of separate git worktrees. This keeps beads state unified and avoids merge complexity.

**This skill REPLACES using-git-worktrees** for concurrent agent work. Beads task granularity provides natural isolation.

## When to Use

- Multiple agents need to work simultaneously
- User asks to "parallelize" or "run agents concurrently"
- Dispatching subagents for independent tasks

**NOT git worktrees.** If you catch yourself creating separate worktrees for agent isolation, STOP. Use beads coordination instead.

## How Agents Coordinate

### 1. Check What Can Parallelize

```bash
bv --robot-plan  # Shows which tasks can run concurrently
```

Tasks without blocking dependencies can run in parallel.

### 2. Claim Work Atomically

Each agent claims exactly ONE task:

```bash
bd update <task-id> --status in_progress --json
```

Other agents see claimed work and pick different tasks.

### 3. Work on Same Branch

- All agents work on the same branch
- Beads tasks are granular → touch different files
- No merge conflicts because different tasks = different code

### 4. Complete and Claim Next

```bash
bd close <task-id> --reason "Completed" --json
bd update <next-task-id> --status in_progress --json
```

## Preventing Conflicts

If two tasks would modify the same file, add a blocking dependency:

```bash
bd dep add <task-a-id> <task-b-id> --type blocks
```

This ensures sequential execution, not concurrent.

**Check before dispatching:**
```bash
bv --robot-plan  # Will show blocking relationships
```

## Quick Reference

| Step | Command | Purpose |
|------|---------|---------|
| Plan | `bv --robot-plan` | See parallelizable tasks |
| Claim | `bd update <id> --status in_progress` | Atomic task ownership |
| Check | `bd list --status in_progress` | See what's claimed |
| Block | `bd dep add <a> <b> --type blocks` | Prevent conflicts |
| Complete | `bd close <id> --reason "Completed"` | Release for next |

## Comparison: Worktrees vs Beads Coordination

| Git Worktrees | Beads Coordination |
|---------------|-------------------|
| Separate .beads/ per worktree | Single source of truth |
| Merge complexity after work | No merges needed |
| Branch management overhead | Same branch |
| State divergence risk | State always unified |
| Setup time per worktree | No setup needed |

## When Beads Coordination Doesn't Work

Use separate checkouts (not worktrees) if tasks truly need:
- Different dependency versions
- Conflicting environment configs
- Isolated test databases

In these cases, coordinate manually and sync .beads/ explicitly.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Worktrees are safer" | Worktrees diverge beads state. That's the danger. |
| "Same branch = conflicts" | Granular beads touch different files. Conflicts are rare. |
| "bv is overhead" | 2 seconds to check parallelization saves hours of conflict resolution. |
| "I'll merge beads later" | Merging beads is painful. Unified state is better. |
| "Isolation feels cleaner" | Beads provides logical isolation without physical duplication. |

## Reference

For bd/bv commands: user's `~/.claude/skills/beads/` and `~/.claude/skills/using-bv/`.
