# Plan 1: CLAUDE.md-Compliant Delete Language Across Skills

## Executive Summary

Multiple skills contain delete-related language that conflicts with CLAUDE.md Rule #1. The TDD skill also added a "stash escape hatch" that defeats TDD's cognitive purpose. This plan standardizes all delete-related language to require explicit human approval while closing cognitive loopholes.

## Problem Statement

1. **TDD skill has escape hatch** - The stash clarification block:
   - Tells agents to use `git stash` (preserves code they should forget)
   - Takes action without asking (violates Rule #1)
   - Defeats TDD's cognitive purpose by keeping pre-TDD code accessible

2. **Multiple skills use imperative delete language** - "Delete it. Start over." without "ask first":
   - `test-driven-development/SKILL.md`
   - `writing-skills/SKILL.md`
   - `testing-skills-with-subagents/SKILL.md`

3. **Test artifact cleanup** - `writing-skills` instructs agents to create test artifacts but provides no guidance on cleanup, leading to `test-baseline.md` files accumulating.

## CLAUDE.md Rule #1 (Absolute)

> "You may NOT delete any file or directory unless I explicitly give the exact command in this session."
> "This includes files you just created (tests, tmp files, scripts, etc.)"
> "If you think something should be removed, stop and ask."

---

## First Principles: Why TDD Requires Fresh Start

The purpose of "discard and start fresh" is NOT punishment. It's **preventing cognitive contamination**.

When you've written implementation code first:
- Your mental model is already biased toward what you built
- Your tests will verify what you implemented, not what should be
- You'll test the happy path you remember, missing edge cases

**The solution isn't "don't look at it" - it's structural prevention.** If code is trivially accessible (stash, reflog), agents WILL rationalize peeking.

---

## Reconciling Rule #1 and TDD

**Rule #1:** "Never delete without permission"
**TDD:** "Delete and start fresh"

**These aren't in conflict. Here's why:**

**Case A: Modified existing files**
- `git checkout <file>` reverts to last commit
- This isn't deletion - the commit is preserved
- NO permission needed - safe and reversible

**Case B: New files (never committed)**
- These only exist in working directory
- "Starting fresh" means deleting them
- Rule #1 DOES apply - ask permission first
- After approval, deletion should be final with NO breadcrumb trail

**Key insight:** Rule #1 requires asking permission. It doesn't require providing escape hatches.

---

## Solution

### Part A: TDD Skill - Remove Escape Hatch + Rule #1 Compliance

**Current state (problematic):**
```markdown
Write code before the test? Discard it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Discard means discard

**Clarification:** "Discard" means revert uncommitted changes, not delete files.

If you've written implementation code before writing tests:
- Use `git checkout <file>` to revert changes
- Or `git stash` if you want to reference later  ← ESCAPE HATCH
- Do NOT use `rm` to delete files
```

**Target state:**
```markdown
Write code before the test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete

**How to delete depends on file status:**

**Modified files (already in git):**
- Use `git checkout <file>` to revert to last commit
- Safe and reversible - no permission needed

**New files (not yet committed):**
- Ask: "[filename] was written before tests. Delete and start fresh with TDD?"
- Wait for explicit approval before deleting
- After approval, delete and start fresh

The point is preventing cognitive contamination. Once discarded, treat the code as if it never existed. Don't try to recover it, reference it, or "just check one thing."

Implement fresh from tests. Period.
```

**Changes:**
1. Revert "Discard" → "Delete" (match upstream language)
2. Remove entire stash clarification block
3. Add "How to delete depends on file status" section with modified/new distinction
4. Keep `bun test` (toolchain preference)

**Locations in TDD skill:**
- Line 37: Change "Discard it. Start over." → "Delete it. Start over."
- Line 43: Change "Discard means discard" → "Delete means delete"
- Lines 45-52: Replace entire clarification block with "How to delete" section
- Line 274 (excuse table): Change "Discard means discard" → "Delete means delete"
- Line 297: Replace entire line with:
  ```markdown
  **All of these mean: Delete and start over with TDD.**

  For modified files: `git checkout <file>`
  For new files: Ask to delete, wait for approval

  Start fresh. Don't peek at the old code.
  ```

### Part B: Writing-Skills - Add "Ask First" + Cleanup Step

**Current state:** Uses "Delete means delete" (matches upstream) but no "ask first".

**Add to each "No exceptions" block:**
```markdown
- **Ask your human partner before deleting**
```

**Locations:**
- Line ~358: After "Delete means delete" in Iron Law section
- Line ~450: After "Delete means delete" in Good/Bad example
- Line ~489: Change "Delete code. Start over" → "Ask to delete the code. Start over"

**Add test artifact cleanup step to Deployment checklist:**

```markdown
- [ ] **Identify test artifacts for cleanup:**
  1. Run `git status` to list untracked files
  2. Identify files created during RED phase testing (typically: `test-baseline.md`, scratch notes, temporary test logs)
  3. **Show your human partner the exact list of files** you propose to remove
  4. **Stop and wait for explicit deletion approval** (CLAUDE.md Rule #1 applies)
  5. Only after receiving explicit approval: delete each named file individually
```

### Part C: Testing-Skills-With-Subagents - Add "Ask First"

**Current state:** Uses "Delete means delete" (matches upstream) but no "ask first".

**Add to "No exceptions" block (~line 201):**
```markdown
- **Ask your human partner before deleting**
```

---

## Files to Edit

| File | Changes |
|------|---------|
| `skills/test-driven-development/SKILL.md` | Lines 37, 43, 45-52, 274, 297: Discard→Delete, remove stash block, add modified/new distinction |
| `skills/writing-skills/SKILL.md` | Lines ~358, ~450, ~489: Add "ask first", update summary line, add cleanup step to deployment checklist |
| `skills/testing-skills-with-subagents/SKILL.md` | Line ~201: Add "ask first" to No exceptions block |

## Files NOT Changed

| File | Reason |
|------|--------|
| `skills/receiving-code-review/SKILL.md` | "DELETE IT" refers to text ("Thanks"), not code files |
| `skills/systematic-debugging/test-pressure-2.md` | Test scenario answer option, not a directive |
| `skills/writing-skills/persuasion-principles.md` | Example text demonstrating pattern, not active directive |

---

## Verification Steps

After edits:

```bash
# Should have zero hits for stash escape hatch
grep -r "git stash" skills/ --include="*.md"

# Should have zero hits for reflog mentions
grep -r "reflog" skills/ --include="*.md"

# All delete-related blocks should have "ask" language
grep -B5 -A5 "Delete means delete" skills/ --include="*.md"
```

---

## Acceptance Criteria

- [ ] TDD skill uses "Delete" language (not "Discard")
- [ ] TDD skill has NO stash references
- [ ] TDD skill has NO reflog references
- [ ] TDD skill has clear modified-vs-new-files distinction
- [ ] TDD skill requires asking for new file deletion only
- [ ] writing-skills has "Ask your human partner before deleting" in all No exceptions blocks
- [ ] writing-skills has test artifact cleanup step with explicit "stop and ask" language
- [ ] testing-skills-with-subagents has "Ask your human partner before deleting" in No exceptions
- [ ] TDD summary line (297) says "Delete and start over" with modified/new clarification below
- [ ] Other skills' summary lines say "Ask to delete the code" (simpler, no modified/new distinction)
- [ ] Consistent messaging across all TDD-related skills

---

## Change Pattern Summary

**For TDD skill - modified vs new files:**
```markdown
**Modified files (already in git):**
- Use `git checkout <file>` to revert to last commit
- Safe and reversible - no permission needed

**New files (not yet committed):**
- Ask: "[filename] was written before tests. Delete and start fresh with TDD?"
- Wait for explicit approval before deleting
```

**For other skills - "No exceptions" blocks:**
```markdown
- **Ask your human partner before deleting**
```

**For TDD summary line (line 297):**
```markdown
# From:
**All of these mean: Discard code (git checkout/stash). Start over with TDD.**

# To:
**All of these mean: Delete and start over with TDD.**

For modified files: `git checkout <file>`
For new files: Ask to delete, wait for approval

Start fresh. Don't peek at the old code.
```

**For other skills' summary lines (writing-skills line ~489):**
```markdown
# From:
All of these mean: Delete code. Start over with TDD.

# To:
All of these mean: Ask to delete the code. Start over with TDD.
```
