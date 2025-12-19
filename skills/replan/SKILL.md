---
name: replan
description: Use when your plan has changed and you need to reconcile bead state - autonomously closes obsolete beads, updates changed ones, and creates new ones by comparing current beads to the new plan
---

# Replan

Reconcile bead state when plans change mid-project. Compares existing beads to the new plan, then closes/updates/creates beads to match.

## When to Use

- You've been executing a plan via beads
- You've realized the approach needs to change
- You've figured out the new direction
- Now you need to clean up bead state

## When NOT to Use

- **Discovered more work during implementation** - use `bd create --deps discovered-from:<id>`
- **Haven't figured out new direction** - use brainstorming skill first
- **Minor scope adjustment to one bead** - just update it directly

## Workflow

### Step 1: Locate the New Plan

Ask: "Is the new plan documented somewhere, or should I work from our conversation?"

### Step 2: Read Both States

```bash
bd list --json                     # All beads
bd list --status completed --json  # Leave these alone
bd list --status in_progress --json
bd list --status pending --json
bd list --status blocked --json
```

Read the new plan (document or conversation context).

### Step 3: Categorize Each Non-Completed Bead

For each bead, decide:

| Decision | When | Action |
|----------|------|--------|
| **Keep** | Bead's work clearly appears in new plan, same approach | No changes |
| **Update** | Work appears but scope/approach changed | Update title, description, notes |
| **Close** | Work not in new plan or superseded | Close with reason explaining what replaced it |

For the new plan:
- Identify work that doesn't map to any existing bead
- Create new beads with appropriate dependencies

### Step 4: Execute Changes

No human approval step - execute based on decision heuristics.

**Order:** Close obsolete beads first, then update, then create new ones.

### Step 5: Report and Verify

Provide diff report:
```
CLOSED:
- <id>: <title> - Reason: <why>

UPDATED:
- <id>: <title> - Changes: <what changed>

CREATED:
- <id>: <title>

UNCHANGED:
- <id>: <title>
```

Verify graph health:
```bash
bv --robot-insights  # Check for cycles, orphans
bd ready --json      # Current actionable state
```

## Decision Heuristics

### Matching Plan Items to Beads

- Match by **intent/outcome**, not exact wording
- "Implement JWT auth" matches "Add authentication layer" if purpose aligns
- When uncertain, favor **update over close** - preserve work history

### When to KEEP

- Bead's described outcome is clearly needed in new plan
- Approach/design still makes sense

### When to UPDATE

- Same general area but scope changed
- Same outcome but different approach
- Update: title, description, notes as needed

### When to CLOSE

- Work no longer needed at all
- Work superseded by fundamentally different approach
- **Close reason must explain what replaced it:** "Superseded by session-based approach" not "No longer needed"

### Creating New Beads

- For plan items with no matching existing bead
- Use `--deps discovered-from:<id>` if related to existing work
- Set priority based on plan's sequencing

## Edge Cases

### In-Progress Bead with Significant Work

Check notes field for COMPLETED items. If closing:
- Preserve what was accomplished in close reason
- Example: "Superseded by session auth. Completed work: JWT dependency setup, token signing logic (not salvageable for new approach)."

### Dependency Chains

When closing a bead that blocks others:
1. Check if blocked beads are also obsolete - cascade close if so
2. If blocked beads are still valid, they need new dependencies or become unblocked
3. Run `bv --robot-insights` after to catch broken graph structure

### Vague or Incomplete Plan

Ask for clarification before proceeding. The plan should be specific enough to map to beads.
