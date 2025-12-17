# Brainstorming Beads Integration - Test Baseline

## Current Behavior

The skill currently:
1. Refines ideas through questioning
2. Presents design in sections
3. Writes to `docs/plans/YYYY-MM-DD-<topic>-design.md`
4. Asks before committing (line 40)
5. References using-git-worktrees and writing-plans for implementation

## Problem

Without beads integration:
- Design document exists but no epic to track implementation
- No persistent link between design and tasks
- Handoff to implementation is informal

## Required Changes

### Add Beads Integration Section

After design document is committed:

1. Create bd epic linking to design doc
2. Set epic description with goal and architecture
3. Offer transition to plan-to-beads skill

### Update Implementation Section

- Reference plan-to-beads instead of writing-plans (for beads workflows)
- Keep using-git-worktrees reference (still useful for isolation)

## Acceptance Criteria

- [ ] Beads Integration section added
- [ ] Creates epic after design approval
- [ ] Links epic to design document
- [ ] Offers transition to plan-to-beads
- [ ] Asks before committing (not auto-commit) - already present
