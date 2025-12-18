# Plan 5: AskUserQuestion Tool Integration (Revised)

## Executive Summary

After first-principles analysis, **only 1 of the 3 proposed use cases should use AskUserQuestion**. The session-start scenarios should remain conversational to allow context-sharing before decisions.

**Revision notes:** This plan targets the current local state of `finishing-a-development-branch`, which includes Step 2.5 (Beads Completion). The revision absorbs Step 2.5's confirmation prompt into Step 3, fixes AskUserQuestion parameter format, and addresses beads state handling on discard.

## Original Proposal Review

| Use Case | Proposed | Decision |
|----------|----------|----------|
| `finishing-a-development-branch` Step 3 | AskUserQuestion with 4 options | **APPROVED** |
| `session-start-with-beads` Step 2a | AskUserQuestion for continue/switch | **REJECTED** |
| `session-start-with-beads` Step 2b | AskUserQuestion for task selection | **REJECTED** |

---

## Analysis: Why Only One Use Case Approved

### finishing-a-development-branch Step 3 - APPROVED

**Current state (with Step 2.5 beads integration):**

Step 2.5 ends with a confirmation prompt, then Step 3 presents 4 text options. This creates double-prompting.

**Why AskUserQuestion fits:**
- Exactly 4 mutually exclusive options with clear outcomes
- User already knows what they want - they just completed work
- No additional context affects the choice
- Timing is AFTER verification (tests pass, beads closed)
- The 4 options ARE exhaustive - no 5th category exists
- AskUserQuestion automatically provides "Other" escape hatch

### session-start-with-beads Step 2a - REJECTED

**Current:** "Continue, or work on something else?"

**Why conversational is better:**
1. **Too early in conversation.** User may want to share context first: "I have a meeting in 30 min" or "Actually, urgent thing came up"
2. **Binary choice doesn't benefit from structure.** Two options isn't meaningfully faster with clicks vs typing
3. **Conversational warmup matters.** Session start sets collaborative tone. Structured menu feels robotic

### session-start-with-beads Step 2b - REJECTED

**Current:** "Present top 3 ready tasks with priorities"

**Why conversational is better:**
1. **"Top 3" is arbitrary.** If 10 tasks exist, showing 3 creates false constraint
2. **Task selection needs context.** User might ask "which is quickest?" or "I need something small before lunch"
3. **Priority algorithm might be wrong.** Graph metrics don't capture user energy, time, mood
4. **Anchoring effect.** Structured options push users toward listed choices

---

## CLAUDE.md Alignment

The rule is:
> "ALWAYS STOP and ask for clarification rather than making assumptions"

This is about **clarification** (uncertainty), not **structured choice** (known options).

**For finishing-a-development-branch:** The 4 options DO cover the decision space completely. This aligns.

**For session-start:** Options DON'T cover the space. "I want to tell you about something first" isn't captured. This would violate the rule.

---

## Refined Specification

### Current State (Local)

The skill currently has:
- **Step 2:** Determine Base Branch
- **Step 2.5:** Beads Completion (closes issues, stages .beads/, asks "Ready to complete? [Proceed / Review]")
- **Step 3:** Present 4 text options
- **Step 4:** Execute Choice

### Changes Required

1. **Absorb Step 2.5 prompt into Step 3** - Remove the intermediate "Proceed / Review" confirmation
2. **Replace Step 3 text with AskUserQuestion** - Use exact tool parameters
3. **Add beads re-open logic to Option 4 (Discard)** - Don't leave orphaned closed issues

### File to Edit

`skills/finishing-a-development-branch/SKILL.md`

---

## Exact AskUserQuestion Specification

### Step 2.5: Beads Completion (Modified)

Remove the confirmation prompt at the end. New Step 2.5:

```markdown
### Step 2.5: Beads Completion

**Before presenting options, close beads:**

1. Close all in_progress tasks for this work:
   ```bash
   bd list --status in_progress --json
   # For each task:
   bd close <task-id> --reason "Complete"
   ```

2. Close the epic (if applicable):
   ```bash
   bd close <epic-id> --reason "Feature complete, ready to merge"
   ```

3. Stage .beads/ for commit:
   ```bash
   git add .beads/
   ```

Proceed directly to Step 3 (no intermediate prompt).
```

### Step 3: Present Options (Replaced)

```markdown
### Step 3: Present Options

Display beads summary, then use AskUserQuestion:

**First, show beads status as informational text:**
```
Beads closed: [task-ids]
Epic [epic-id] closed (if applicable)
.beads/ staged for commit
```

**Then invoke AskUserQuestion with these exact parameters:**

```json
{
  "questions": [{
    "question": "How should I complete this branch?",
    "header": "Action",
    "multiSelect": false,
    "options": [
      {
        "label": "Merge locally",
        "description": "Merge to {base-branch}, delete feature branch"
      },
      {
        "label": "Create PR",
        "description": "Push and open pull request for review"
      },
      {
        "label": "Keep as-is",
        "description": "Leave branch, I'll handle it later"
      },
      {
        "label": "Discard",
        "description": "Delete branch and commits, re-open beads (confirmation required)"
      }
    ]
  }]
}
```

**Dynamic substitution:** Replace `{base-branch}` with the actual branch name determined in Step 2.

**Fallback:** If user responds with text (e.g., "1" or "merge") instead of clicking, interpret as that choice.
```

### Step 4, Option 4: Discard (Modified)

Add beads re-open before branch deletion:

```markdown
#### Option 4: Discard

**Confirm first:**
```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Beads will be re-opened (work still needs doing).

Type 'discard' to confirm.
```

Wait for exact confirmation.

If confirmed:
```bash
# Re-open beads with context
bd list --status closed --json  # Find recently closed issues for this work
# For each:
bd update <id> --status ready
bd update <id> --notes "DISCARDED: Branch deleted. Previous implementation abandoned."

# Unstage .beads/ changes and restore
git restore --staged .beads/
git checkout .beads/

# Delete branch
git checkout <base-branch>
git branch -D <feature-branch>
```

Then: Cleanup worktree (Step 5)
```

---

## Files NOT to Edit

- `skills/session-start-with-beads/SKILL.md` - Keep conversational flow

---

## Risk Mitigation

### Tool parameter changes
**Mitigation:** Exact JSON spec documented. If AskUserQuestion schema changes, update the JSON accordingly.

### Options don't fit situation
**Analysis:** The 4 options ARE exhaustive for a completed branch:
1. Merge (integrate)
2. PR (defer integration)
3. Keep (defer decision)
4. Discard (abandon)

AskUserQuestion automatically provides "Other" for unexpected cases.

### Over-structuring
**Mitigation:** Apply AskUserQuestion ONLY where options are mutually exclusive AND exhaustive AND timing is appropriate AND user is ready to decide.

### Discard leaves orphaned beads
**Mitigation:** Re-open beads on discard with note explaining what happened. User can manually close with "Won't fix" if they truly don't want the work.

### Double-prompting UX
**Mitigation:** Absorbed Step 2.5 confirmation into Step 3. Single decision point after beads status display.

---

## Testing Approach

1. **Manual verification:** Invoke skill in real branch completion scenario
2. **Check tool invocation:** Verify agent produces correct AskUserQuestion JSON parameters
3. **Verify header:** Confirm "Action" header appears (max 12 chars)
4. **Test dynamic substitution:** Confirm `{base-branch}` is replaced with actual branch name
5. **Test text fallback:** Type "1" or "merge" instead of clicking, verify correct interpretation
6. **Test Option 4 beads handling:** Discard a branch, verify beads are re-opened with note
7. **User feedback:** Complete 3 real branch completions, assess UX

---

## Acceptance Criteria

- [ ] Step 2.5 no longer has intermediate "Proceed / Review" prompt
- [ ] Step 3 shows beads summary as text, then invokes AskUserQuestion
- [ ] AskUserQuestion uses exact JSON format with all required fields (`question`, `header`, `multiSelect`, `options`)
- [ ] `header` is "Action" (within 12 char limit)
- [ ] `multiSelect` is `false`
- [ ] Dynamic value `{base-branch}` substitution documented
- [ ] Option 4 description mentions beads re-open
- [ ] Option 4 implementation re-opens beads with note before deleting branch
- [ ] Fallback for text responses documented
- [ ] `session-start-with-beads` remains conversational (NO changes)
- [ ] No other skills modified
