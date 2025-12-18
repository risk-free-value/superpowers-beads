# Plan 2: Remove Deprecated Skills

## Executive Summary

This task requires **directory deletion**, which means CLAUDE.md Rule #1 applies absolutely. The plan must be structured with mandatory human approval checkpoints. Additionally, investigation revealed **hidden dependencies** that must be updated BEFORE deletion.

## Problem Statement

Two skills have deprecation notices pointing to beads-integrated replacements:
1. **`writing-plans`** → superseded by `plan-to-beads`
2. **`using-git-worktrees`** → superseded by `multi-agent-coordination`

**Beads is a hard requirement for Superpowers users.** Keeping deprecated alternatives adds confusion and increases the chance agents accidentally use the wrong skill.

## Critical CLAUDE.md Alignment

**Rule #1 (Absolute):**
> "You may NOT delete any file or directory unless I explicitly give the exact command in this session."
> "If you think something should be removed, stop and ask. You must receive clear written approval before any deletion command is even proposed."

**This plan is a PROPOSAL, not authorization.** The deletion step requires explicit in-session approval from Will.

---

## Hidden Dependencies Discovered

Investigation revealed references that would break after deletion:

| File | Reference Type | Impact if Deleted Without Update |
|------|---------------|----------------------------------|
| `commands/write-plan.md` | **Direct invocation** | Command becomes broken |
| `skills/brainstorming/SKILL.md` | **Non-beads fallback path** | References deleted skills |
| `skills/finishing-a-development-branch/SKILL.md` | **"Pairs with" documentation** | References deleted skill |
| `README.md` | Listed as active skill | Documentation describes deleted skills |

**Safe to preserve (historical context):**
- `skills/plan-to-beads/SKILL.md` - contrast language ("REPLACES...")
- `skills/multi-agent-coordination/SKILL.md` - contrast language
- `docs/plans/*.md` - design documents
- `RELEASE-NOTES.md` - changelog

---

## Phased Execution Plan

### PHASE 1: Pre-Deletion Verification

**Objective:** Confirm prerequisites are met before any destructive action.

#### Step 1.1: Verify Replacement Skills Exist
```bash
ls -la skills/plan-to-beads/SKILL.md
ls -la skills/multi-agent-coordination/SKILL.md
```
**Expected:** Both files exist and are non-empty.

#### Step 1.2: Verify All References Identified
```bash
grep -r "writing-plans" skills/ README.md commands/ --include="*.md" | grep -v "skills/writing-plans/"
grep -r "using-git-worktrees" skills/ README.md commands/ --include="*.md" | grep -v "skills/using-git-worktrees/"
```

---

### PHASE 2: Pre-Deletion Updates

**Objective:** Update all breaking references BEFORE deletion.

#### Step 2.1: Rename `commands/write-plan.md` to `commands/plan-to-beads.md`

**Rationale:** The command name should match the skill it invokes. `/write-plan` suggests creating a plan from scratch, but `plan-to-beads` converts an existing design to beads.

**Delete:** `commands/write-plan.md`

**Create:** `commands/plan-to-beads.md` with content:
```markdown
---
description: Create beads from approved design
---

Use the plan-to-beads skill exactly as written
```

#### Step 2.2: Update `skills/brainstorming/SKILL.md`

**Rationale:** Since beads is required, non-beads paths are dead code. Delete them rather than maintaining parallel paths.

**Delete lines 64-67** (the entire "Implementation (if continuing without beads):" section):
```markdown
**Implementation (if continuing without beads):**
- Ask: "Ready to set up for implementation?"
- Use superpowers:using-git-worktrees to create isolated workspace
- Use superpowers:writing-plans to create detailed implementation plan
```

The beads path already exists at lines 42-62 ("Beads Integration:").

#### Step 2.3: Update `skills/finishing-a-development-branch/SKILL.md`

**Rationale:** The "Pairs with" section documents a workflow pairing that no longer exists. `multi-agent-coordination` doesn't create worktrees, so there's no equivalent pairing. The skill's Step 5 still handles worktree cleanup generically if worktrees exist.

**Delete lines 238-239** (the entire "Pairs with:" section):
```markdown
**Pairs with:**
- **using-git-worktrees** - Cleans up worktree created by that skill
```

#### Step 2.4: Update `README.md`

Multiple updates required:

**2.4a - Line 57 (Verify Installation section):**

Change:
```
# /superpowers:write-plan - Create implementation plan
```

To:
```
# /superpowers:plan-to-beads - Create beads from approved design
```

**2.4b - Lines 82-87 (The Basic Workflow section):**

Current steps 2-3:
```markdown
2. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

3. **writing-plans** - Activates with approved design. Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps.
```

Replace with:
```markdown
2. **plan-to-beads** - Activates after design approval. Converts design document into granular, self-documenting beads with full dependency structure. Each bead contains WHAT/WHY/HOW/CONTEXT/ACCEPTANCE.

3. **multi-agent-coordination** - Coordinates concurrent agents via beads status instead of separate git worktrees. Keeps state unified, avoids merge complexity.
```

**2.4c - Lines 133-136 (Deprecated Skills section):**

Delete entire section:
```markdown
### Deprecated Skills (for beads workflows)

- **writing-plans** - Use plan-to-beads instead
- **using-git-worktrees** - Use multi-agent-coordination instead
```

**2.4d - Lines 155 and 160 (What's Inside / Collaboration section):**

Remove these two lines from the Collaboration list:
```markdown
- **writing-plans** - Detailed implementation plans
- **using-git-worktrees** - Parallel development branches
```

---

### PHASE 3: Human Approval Checkpoint (MANDATORY)

**Agent MUST present this summary and STOP:**

```
DELETION APPROVAL REQUEST

I have completed pre-deletion updates:
- Renamed commands/write-plan.md to commands/plan-to-beads.md
- Deleted non-beads fallback from skills/brainstorming/SKILL.md (lines 64-67)
- Deleted "Pairs with" section from skills/finishing-a-development-branch/SKILL.md (lines 238-239)
- Updated README.md:
  - Changed /superpowers:write-plan to /superpowers:plan-to-beads in verification section
  - Rewrote Basic Workflow steps 2-3 for beads
  - Removed "Deprecated Skills" section
  - Removed writing-plans and using-git-worktrees from Collaboration list

Ready to delete:
1. skills/writing-plans/ (directory containing SKILL.md)
2. skills/using-git-worktrees/ (directory containing SKILL.md)

These skills are superseded by plan-to-beads and multi-agent-coordination respectively.
Both replacement skills verified to exist and be complete.

Will, please provide explicit authorization to delete these directories.
Example: "Approved. Delete skills/writing-plans/ and skills/using-git-worktrees/"
```

**Agent MUST wait for explicit written approval before proceeding.**

---

### PHASE 4: Execution (Only After Approval)

**Only execute if Will provides explicit in-session authorization.**

#### Step 4.1: Record Authorization

Before executing, agent records:
- Exact text of Will's authorization
- What will be deleted

#### Step 4.2: Execute Deletion

```bash
rm -rf skills/writing-plans/
rm -rf skills/using-git-worktrees/
```

#### Step 4.3: Verify Deletion

```bash
ls skills/writing-plans 2>&1   # Should show "No such file or directory"
ls skills/using-git-worktrees 2>&1   # Should show "No such file or directory"
```

---

### PHASE 5: Post-Deletion Validation

#### Step 5.1: Check for Dangling References

```bash
grep -r "writing-plans" skills/ README.md commands/ --include="*.md"
grep -r "using-git-worktrees" skills/ README.md commands/ --include="*.md"
```

**Expected:** Only historical/contrast references remain (in plan-to-beads and multi-agent-coordination SKILL.md files).

#### Step 5.2: Commit Changes

```bash
git add -A
git status  # Review all changes
```

**Agent asks Will:** "Ready to commit deletion and updates. Proceed?"

---

## Rollback Plan

If something breaks after deletion:

```bash
git reflog  # Find commit before deletion
git checkout <commit-hash> -- skills/writing-plans/ skills/using-git-worktrees/
```

---

## Acceptance Criteria

- [ ] Replacement skills (`plan-to-beads`, `multi-agent-coordination`) verified to exist
- [ ] All breaking references updated BEFORE deletion:
  - [ ] `commands/write-plan.md` renamed to `commands/plan-to-beads.md`
  - [ ] `skills/brainstorming/SKILL.md` non-beads path deleted (lines 64-67)
  - [ ] `skills/finishing-a-development-branch/SKILL.md` "Pairs with" deleted (lines 238-239)
  - [ ] `README.md` updated (4 changes: verification, workflow, deprecated section, collaboration list)
- [ ] Explicit in-session approval received from Will
- [ ] Authorization recorded with exact text
- [ ] `skills/writing-plans/` removed
- [ ] `skills/using-git-worktrees/` removed
- [ ] Post-deletion grep confirms no unexpected dangling references
- [ ] Changes committed with `.beads/` state
