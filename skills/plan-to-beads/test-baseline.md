# Baseline Test: Plan Conversion Without plan-to-beads Skill

## Scenario

You are an AI assistant with superpowers skills (including brainstorming) and beads (bd CLI).
Brainstorming has JUST completed. You have a design document at `docs/plans/2025-12-17-auth-feature-design.md`.

User message: "Great, let's turn this design into an implementation plan"

## Pressure Points

1. **Habit**: writing-plans skill exists and creates markdown implementation plans
2. **Convenience**: Markdown plans are familiar, beads require multiple bd commands
3. **Completeness illusion**: Vague beads feel "done" faster than self-documenting ones
4. **Dependency complexity**: Skipping dependency structure seems simpler

## Expected Failure Modes (Baseline)

Without plan-to-beads skill, agent likely:

1. **Creates markdown plan**: Uses writing-plans skill to create `docs/plans/...-implementation.md`
2. **Creates vague beads**: `bd create "Implement auth"` without WHAT/WHY/HOW/CONTEXT/ACCEPTANCE
3. **Skips dependency structure**: No `bd dep add` commands, no parent-child relationships
4. **Doesn't verify with bv**: Doesn't run `bv --robot-plan` to check execution order
5. **Doesn't quality check**: Doesn't ask "could a fresh agent implement this?"

## Test Command

Run a subagent with superpowers but WITHOUT plan-to-beads:

```
Task subagent with:
- Access to superpowers skills (except plan-to-beads)
- Working directory with .beads/
- Design doc exists at docs/plans/2025-12-17-auth-feature-design.md
- User prompt: "Great, let's turn this design into an implementation plan"

Document:
- Did agent create markdown plan or beads?
- If beads, did descriptions include WHAT/WHY/HOW/CONTEXT/ACCEPTANCE?
- Did agent add dependency structure?
- Did agent verify with bv?
- Did agent quality check for self-contained beads?
```

## Success Criteria for Skill

After skill is written, same scenario should result in:
1. Agent creates beads, NOT markdown implementation plan
2. Each bead has full WHAT/WHY/HOW/CONTEXT/ACCEPTANCE structure
3. Agent creates epic first, links tasks as children
4. Agent adds blocking dependencies where needed
5. Agent runs bv --robot-plan to verify
6. Agent asks quality check question for each bead
