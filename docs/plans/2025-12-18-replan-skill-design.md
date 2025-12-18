# Replan Skill Design

## Overview

A skill for reconciling bead state when your plan changes mid-project. Handles the mechanics of triaging existing beads and creating new ones - assumes you already know the new direction.

## Skill Metadata

- **Name:** `replan`
- **Description:** Use when your plan has changed and you need to reconcile bead state - autonomously closes obsolete beads, updates changed ones, and creates new ones based on comparing current beads to the new plan
- **Location:** Standalone skill in superpowers collection

## When to Use

- You've been executing a plan via beads
- You've realized the approach needs to change
- You've figured out the new direction (via brainstorming or otherwise)
- Now you need to clean up bead state to match the new reality

## When NOT to Use

- Just discovered more work during implementation → use normal `bd create --deps discovered-from`
- Haven't figured out the new direction yet → use brainstorming skill first
- Minor scope adjustment to one bead → just update it directly

## Workflow

### Step 1 - Locate the New Plan

Ask: "Is the new plan documented somewhere, or should I work from our conversation?"

- If documented → read the plan file
- If conversation → work from context already discussed

### Step 2 - Read Both States

Read the new plan (document or conversation context).

Read current bead state:
```bash
bd list --json                    # All beads
bd list --status completed --json # What's done (leave alone)
bd list --status in_progress --json
bd list --status pending --json
bd list --status blocked --json
```

### Step 3 - Agent Reconciles Autonomously

For each non-completed bead, decide:
- **Keep** - Bead's work clearly appears in the new plan
- **Update** - Work appears but scope/approach changed
- **Close** - Work doesn't appear in new plan or is superseded

For the new plan:
- Identify work that doesn't map to any existing bead → create new beads

### Step 4 - Execute All Changes

No human approval step - execute based on decision heuristics.

### Step 5 - Report and Verify

- Diff report: what was closed/updated/created and why
- `bv --robot-plan` + `bv --robot-insights` for graph health
- `bd ready --json` for current state

## Decision Heuristics

### Matching Plan Items to Beads

- Match by intent/outcome, not exact wording
- A bead "Implement JWT auth" matches plan item "Add authentication layer" if the purpose aligns
- When uncertain, favor **update** over **close** - preserve work history

### When to KEEP (no changes)

- Bead's described outcome is clearly needed in new plan
- Approach/design in bead still makes sense

### When to UPDATE

- Same general area but scope changed (broader, narrower, different boundaries)
- Same outcome but different approach/design
- Update the relevant fields: description, design, acceptance criteria

### When to CLOSE

- Work is no longer needed at all
- Work is superseded by a fundamentally different approach (not just refined)
- Reason should explain what replaced it: "Superseded by event-driven approach" not just "No longer needed"

### Creating New Beads

- For plan items with no matching existing bead
- Use appropriate dependency types if they relate to existing work
- Set priority based on plan's sequencing/importance

## Edge Cases and Failure Modes

### Vague or Incomplete Plan

Ask for clarification before proceeding. Don't guess at intent - the plan should be specific enough to map to beads.

### In-Progress Bead with Significant Work

- Check notes field for what's been done
- If partial work is salvageable, update the bead to reflect new scope, preserve COMPLETED items in notes
- If work is truly obsolete, close it - but note what was completed in the close reason

### Dependency Chains

- When closing a bead that blocks others, check if blocked beads are also obsolete
- Don't leave orphaned blockers - cascade the triage logic
- Use `bv --robot-insights` to catch any broken graph structure after

### Uncertain About a Specific Bead

- Err toward update over close (preserve history)
- Err toward keep over update (minimize churn)
- If genuinely ambiguous, ask - but this should be rare if the plan is clear
