# Baseline Test: Checkpointing Without beads-checkpoint Skill

## Scenario

You are an AI assistant with superpowers and beads. You've been working for a while.
Token usage is approaching 70%. You have in-progress beads.

User message: "I need to step away, let's pause here"

## Pressure Points

1. **Urgency**: User wants to stop, feels wrong to delay with checkpointing
2. **Optimism**: "I'll remember where we were" / "The summary will be fine"
3. **Vagueness habit**: Writing "made good progress" instead of specifics
4. **Commit friction**: Skipping commit because "it's just notes"

## Expected Failure Modes (Baseline)

Without beads-checkpoint skill, agent likely:

1. **Skips checkpoint entirely**: "Sure, see you later!" - no notes written
2. **Writes vague notes**: "COMPLETED: worked on feature. NEXT: continue."
3. **Waits for compaction**: Only checkpoints when forced by precompact hook
4. **Forgets to commit**: Updates notes but doesn't commit .beads/
5. **No quality check**: Doesn't ask "could a fresh agent continue?"

## Test Command

Run subagent with superpowers but WITHOUT beads-checkpoint:

```
Task subagent with:
- Access to superpowers skills (except beads-checkpoint)
- Working directory with .beads/ and in-progress issues
- Simulated 70% token usage context
- User prompt: "I need to step away, let's pause here"

Document:
- Did agent proactively checkpoint?
- Were notes specific or vague?
- Did agent commit beads state?
- Did agent ask quality check question?
```

## Success Criteria for Skill

After skill is written, same scenario should result in:
1. Agent recognizes checkpoint trigger (user stepping away)
2. Agent writes specific structured notes (COMPLETED/IN PROGRESS/NEXT/BLOCKERS)
3. Agent asks quality check: "Could a fresh agent continue with these notes?"
4. Agent asks to commit beads state
