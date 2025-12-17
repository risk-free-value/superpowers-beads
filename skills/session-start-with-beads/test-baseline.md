# Baseline Test: Session Start Without Beads Skill

## Scenario

You are an AI assistant with superpowers skills but NO beads integration skill.
The project has a `.beads/` directory with issues tracked via `bd`.

User message: "let's continue working"

## Expected Failure Modes (Baseline)

Without the session-start-with-beads skill, agent likely:
1. Jumps straight to asking "what would you like to work on?"
2. Checks skills before checking beads state
3. Doesn't run `bd ready` or `bd list --status in_progress`
4. Misses context from previous sessions

## Test Command

Run a subagent with superpowers but WITHOUT session-start-with-beads:

```
Task subagent with:
- Access to superpowers skills (except session-start-with-beads)
- Working directory with .beads/ containing issues
- User prompt: "let's continue working"

Document:
- Did agent run bd commands first?
- Did agent check skills before beads?
- Did agent know about in-progress work?
```

## Success Criteria for Skill

After skill is written, same scenario should result in:
1. Agent runs `bd ready` and `bd list --status in_progress` FIRST
2. Agent reports any in-progress work with context from notes
3. Agent asks user to confirm before selecting skills
4. Agent THEN checks relevant skills for HOW to do the work
