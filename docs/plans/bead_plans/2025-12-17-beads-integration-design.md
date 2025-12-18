# Superpowers-Beads Integration Design

> **For Implementation:** This design document should be converted to granular beads for execution. See `.beads/` directory for implementation tasks.

**Created:** 2025-12-17
**Status:** Approved for implementation

---

## Executive Summary

Fork Superpowers and modify it to use Beads for persistent task tracking and work direction, while preserving the workflow discipline (brainstorming, TDD, code review).

**Core insight:** Superpowers provides workflow discipline (HOW to work), Beads provides work direction (WHAT to work on). Integration teaches Superpowers that work can span sessions and there's a strategic layer above workflows.

---

## Background

### What is Superpowers?

A Claude Code plugin (`obra/superpowers`) providing composable "skills" for software development workflows:
- **21 skills** covering brainstorming, TDD, debugging, code review, planning
- **3 slash commands:** `/brainstorm`, `/write-plan`, `/execute-plan`
- **1 subagent type:** `superpowers:code-reviewer`

**Core workflow:**
```
brainstorming → using-git-worktrees → writing-plans → executing-plans → code-review → finishing
```

**Key files location:** `~/.claude/plugins/cache/superpowers-marketplace/superpowers/3.6.2/`

### What is Beads?

A graph-based issue tracker (`steveyegge/beads`) designed for AI agent workflows:
- **`bd` CLI:** CRUD operations on issues stored in `.beads/beads.jsonl`
- **Structured notes:** `COMPLETED/IN PROGRESS/NEXT/BLOCKERS` format survives compaction
- **Dependency graph:** `blocks`, `related`, `parent-child`, `discovered-from`
- **Session handoff:** Issues persist across context compaction

### What is Beads Viewer?

Terminal UI and graph analysis (`Dicklesworthstone/beads_viewer`):
- `bv --robot-plan` - Parallelizable execution plan (instant)
- `bv --robot-priority` - Priority suggestions with reasoning (instant)
- `bv --robot-insights` - Graph metrics: PageRank, betweenness, cycles (~500ms)

### Why Integrate?

| Superpowers Alone | With Beads Integration |
|-------------------|------------------------|
| Each session is isolated | Work persists across sessions |
| TodoWrite lost after session | Beads survives compaction |
| Manual task selection | bv suggests what to work on |
| Markdown plans for tracking | Beads for tracking, markdown for design only |
| Git worktrees for isolation | Beads coordination for multi-agent |

---

## Architecture

### Layer Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    STRATEGIC LAYER (Beads)                      │
│  - What work exists (bd list)                                   │
│  - What's ready (bd ready)                                      │
│  - What's in progress (bd list --status in_progress)            │
│  - Priority/order (bv --robot-plan)                             │
│  - Acceptance criteria (bd show <id>)                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   WORKFLOW LAYER (Superpowers)                  │
│  - HOW to brainstorm (brainstorming skill)                      │
│  - HOW to create beads from plan (plan-to-beads skill)          │
│  - HOW to execute (executing-plans, subagent-driven-dev)        │
│  - HOW to test (test-driven-development)                        │
│  - HOW to review (requesting-code-review)                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  TACTICAL LAYER (TodoWrite)                     │
│  - Current session's checklist steps                            │
│  - Visible progress to user                                     │
│  - Ephemeral, lost after session                                │
└─────────────────────────────────────────────────────────────────┘
```

### New Workflow

```
Session Start → bd ready/in_progress → bv --robot-plan suggestions
                         ↓
              Skill selection (HOW to do the work)
                         ↓
              Brainstorming → design.md → bd epic
                         ↓
              plan-to-beads → granular bd tasks with dependencies
                         ↓
              Execution driven by bd tasks (NOT markdown plans)
                         ↓
              Code review checks bd acceptance criteria
                         ↓
              Close bd issues, commit .beads/ with code
```

---

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Commit policy | Ask before committing | Respect CLAUDE.md preference for explicit control |
| Toolchain | bun (JS), uv (Python) | CLAUDE.md requirements |
| Plan documents | Design only, no implementation plans | Beads replaces implementation tracking |
| Git worktrees | Deprecate | Beads provides coordination, worktrees complicate state |
| Sub-skill loading | Reference only, don't require | Let agent use CLAUDE.md context, load skill if needed |
| Checkpointing | Proactive at 70-85%, hook as safety net | Better notes with more context available |
| Beads granularity | One task per implementation step | More granular for better tracking |
| bv suggestions | Inform, don't replace user choice | Present suggestions, user decides |

---

## TodoWrite vs Beads Clarification

**Both coexist at different timescales:**

| Aspect | TodoWrite | Beads |
|--------|-----------|-------|
| Persistence | Session only | Survives compaction |
| Timescale | Minutes (this hour) | Days/weeks |
| Purpose | Visible checklist | Strategic tracking |
| Granularity | Fine execution steps | Meaningful work units |

**Pattern:**
1. Beads tracks the work item strategically
2. TodoWrite tracks current session's tactical steps
3. Update beads notes at milestones (70% tokens, task complete, blockers)
4. Close beads issue when work complete

**Don't duplicate:** TodoWrite steps don't need to match beads acceptance criteria 1:1.

---

## Compatibility Requirements (CLAUDE.md)

The user's CLAUDE.md has specific requirements that Superpowers skills must respect:

### Toolchain
- **JavaScript/TypeScript:** Use `bun` (never npm, yarn, pnpm)
- **Python:** Use `uv` (never pip, venv, virtualenv, pipenv)

### Behavioral
- **Commits:** Only create commits when user explicitly approves
- **File deletion:** Never delete files without explicit permission
- **TDD "delete":** Means "discard uncommitted work" not "delete files"

### Skills Reference
- User has `~/.claude/skills/beads/` with full beads documentation
- User has `~/.claude/skills/using-bv/` with bv documentation
- User's CLAUDE.md has bd/bv command reference
- Skills should reference these, not require loading them every time

---

## Files to Create

### New Skills

| File | Purpose |
|------|---------|
| `skills/session-start-with-beads/SKILL.md` | Session start protocol checking bd before skills |
| `skills/plan-to-beads/SKILL.md` | Convert design to granular, self-documenting beads |
| `skills/beads-checkpoint/SKILL.md` | Checkpointing protocol for compaction survival |
| `skills/multi-agent-coordination/SKILL.md` | Replace git-worktrees for multi-agent work |

### Modified Skills

| File | Changes |
|------|---------|
| `skills/using-superpowers/SKILL.md` | Add bd check to protocol, clarify TodoWrite vs Beads |
| `skills/brainstorming/SKILL.md` | Create bd epic after design, ask before commit |
| `skills/writing-plans/SKILL.md` | Add deprecation notice, redirect to plan-to-beads |
| `skills/executing-plans/SKILL.md` | Read from beads, update bd status, ask before commit |
| `skills/subagent-driven-development/SKILL.md` | Pass bd acceptance to subagents, update bd status |
| `skills/using-git-worktrees/SKILL.md` | Add deprecation notice |
| `skills/verification-before-completion/SKILL.md` | Verify bd acceptance criteria |
| `skills/finishing-a-development-branch/SKILL.md` | Close bd issues, commit .beads/ |
| `skills/test-driven-development/SKILL.md` | Fix toolchain (bun), clarify discard vs delete |

### Modified Agents

| File | Changes |
|------|---------|
| `agents/code-reviewer.md` | Check bd acceptance criteria, report beads status |

### Documentation

| File | Changes |
|------|---------|
| `README.md` | Add beads integration section, update workflow |

---

## Detailed Specifications

### session-start-with-beads Skill

```markdown
Purpose: Unified session start that checks beads BEFORE skill selection.

Protocol:
1. Check bd state:
   - bd ready --json
   - bd list --status in_progress --json

2. If in_progress exists:
   - bd show <id> to read notes
   - Report: "Continuing [title]. Last: [COMPLETED]. Next: [NEXT]"
   - Ask: "Continue, or different work?"

3. If nothing in_progress:
   - bv --robot-plan for suggestions
   - Present top items to user
   - Ask: "Which to work on?"

4. THEN check skills for HOW to do chosen work

Reference (don't require): ~/.claude/skills/beads/, ~/.claude/skills/using-bv/
```

### plan-to-beads Skill

```markdown
Purpose: Convert approved design into granular, self-documenting beads.

Process:
1. Read the design document - understand goals, architecture, constraints

2. Create epic:
   bd create "Feature: [name]" -t epic -p [priority] --json
   bd update <epic-id> --description "[Full context]
   Goal: [what and why]
   Architecture: [key decisions]
   Design doc: docs/plans/YYYY-MM-DD-<topic>-design.md
   Success criteria: [what done looks like]"

3. Decompose into granular tasks - each must be self-contained

4. Task description format:
   WHAT: Specific deliverable
   WHY: How it serves the goal
   HOW: Implementation approach
   CONTEXT: Background, constraints, considerations
   ACCEPTANCE:
   - [ ] Criterion 1
   - [ ] Criterion 2

5. Add dependency structure:
   - parent-child: epic to tasks
   - blocks: where order matters
   - discovered-from: for tasks found during work

6. Verify with bv --robot-plan

7. Quality check: "Could fresh agent implement from this bead alone?"
```

### beads-checkpoint Skill

```markdown
Purpose: Formalize checkpointing for compaction survival.

Triggers (in order of preference):
- 70% token usage: Proactive checkpoint (best quality notes)
- 85% token usage: Warning, checkpoint immediately
- 90% token usage: Emergency auto-checkpoint
- Precompact hook: Safety net (notes may be degraded)

Protocol:
1. bd list --status in_progress --json
2. For each, write structured notes:
   bd update <id> --notes "COMPLETED: [specific deliverables]
   IN PROGRESS: [current state + next step]
   NEXT: [concrete action]
   BLOCKERS: [what's preventing progress]
   KEY DECISIONS: [important context with rationale]"
3. git add .beads/ && git commit -m "chore: checkpoint beads state"

Quality check: "If fresh agent started with ONLY these notes, could they continue?"
```

### multi-agent-coordination Skill

```markdown
Purpose: Replace git-worktrees for multi-agent concurrent work.

How agents coordinate:
1. Check available: bd ready --json, bd list --status in_progress --json
2. Claim atomically: bd update <task-id> --status in_progress
3. Other agents see claimed work, pick different tasks
4. Work on same branch (beads provides isolation)

Preventing conflicts:
- If two tasks modify same file: bd dep add <a> <b> --type blocks
- Granular tasks naturally touch different code

Advantages over worktrees:
- No merge complexity
- Shared beads state (single source of truth)
- Simpler mental model
- One branch to test in CI
```

### code-reviewer.md Changes

```markdown
Add to review process:

7. **Beads Acceptance Check** (if issue ID provided):
   - Run bd show <issue-id>
   - Verify each acceptance criterion met
   - Report status in review output

8. **Review Output Addition:**
   ## Beads Status
   - Acceptance criteria: X/Y met
   - [ ] Criterion 1: ✓ Met
   - [ ] Criterion 2: ✗ Not met - [reason]
   - Recommendation: [Close issue / Continue work / Create follow-up]
```

### Toolchain Fixes

All skills using test commands:
- `npm test` → `bun test`
- `npm install` → `bun install`
- `pytest` → `uv run pytest`
- Add note: "Adjust commands for your project's toolchain"

### Commit Policy Fixes

All skills that commit:
- Replace auto-commit with "Ready to commit? [show changes]"
- Wait for user approval
- Batch where sensible

### TDD Clarification

Change "Delete code written before tests" to:
- "Discard uncommitted work written before tests"
- Add: "Use git checkout to revert, not rm"
- Clarify: This is about uncommitted code, not deleting files

---

## Dependency Graph

```
Phase 1: Fork/Setup
    │
    ├──▶ 1.1 Fork repository
    └──▶ 1.2 Create development branch
              │
              ▼
Phase 2: Core Infrastructure
    │
    ├──▶ 2.1 Create session-start-with-beads ─────┐
    ├──▶ 2.2 Modify using-superpowers ◀───────────┤
    └──▶ 2.3 Create beads-checkpoint              │
              │                                    │
              ▼                                    │
Phase 3: Plan → Beads Workflow                    │
    │                                              │
    ├──▶ 3.1 Create plan-to-beads ◀───────────────┤
    ├──▶ 3.2 Modify brainstorming                 │
    └──▶ 3.3 Deprecate writing-plans              │
              │                                    │
              ▼                                    │
Phase 4: Execution Workflow                       │
    │                                              │
    ├──▶ 4.1 Modify executing-plans ◀─────────────┤
    ├──▶ 4.2 Modify subagent-driven-development   │
    ├──▶ 4.3 Create multi-agent-coordination      │
    └──▶ 4.4 Deprecate using-git-worktrees        │
              │                                    │
              ▼                                    │
Phase 5: Review/Completion                        │
    │                                              │
    ├──▶ 5.1 Modify code-reviewer ◀───────────────┘
    ├──▶ 5.2 Modify verification-before-completion
    └──▶ 5.3 Modify finishing-a-development-branch
              │
              ▼
Phase 6: Compatibility (parallel with 3-5)
    │
    ├──▶ 6.1 Fix toolchain references
    ├──▶ 6.2 Fix commit policy
    └──▶ 6.3 Clarify TDD discard
              │
              ▼
Phase 7: Testing/Documentation
    │
    ├──▶ 7.1 End-to-end test
    └──▶ 7.2 Update README
```

---

## Implementation Notes

### Getting Started

1. Fork `obra/superpowers` to your GitHub
2. Clone to `~/projects/Superpowers/`
3. The original skills are at `~/.claude/plugins/cache/superpowers-marketplace/superpowers/3.6.2/`
4. Copy structure to your fork, then modify

### Testing Changes

Test each skill modification by:
1. Installing your fork as a local plugin
2. Running the workflow it affects
3. Verifying beads state updates correctly

### Reference Materials

- User's beads skill: `~/.claude/skills/beads/SKILL.md` and `references/`
- User's bv skill: `~/.claude/skills/using-bv/SKILL.md`
- User's CLAUDE.md: `~/.claude/CLAUDE.md`
- Original Superpowers: `~/.claude/plugins/cache/superpowers-marketplace/superpowers/3.6.2/`

---

## Success Criteria

The integration is complete when:

- [ ] Session start checks bd before skill selection
- [ ] Brainstorming creates bd epic after design approval
- [ ] Design converts to granular beads (not markdown implementation plans)
- [ ] Execution is driven by bd tasks
- [ ] Code review verifies bd acceptance criteria
- [ ] All skills respect CLAUDE.md (bun, uv, ask before commit)
- [ ] Checkpointing happens proactively at 70%+ tokens
- [ ] Multi-agent coordination works via beads (no worktrees)
- [ ] End-to-end workflow tested successfully
