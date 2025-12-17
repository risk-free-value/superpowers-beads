# Baseline Test: Multi-Agent Work Without multi-agent-coordination Skill

## Scenario

You are an AI assistant with superpowers and beads. User wants to run multiple agents concurrently on different tasks.

User message: "I want to parallelize this - can we have multiple agents working at once?"

## Pressure Points

1. **Familiarity**: using-git-worktrees is the existing pattern for isolation
2. **Isolation instinct**: "Separate branches = no conflicts" feels safe
3. **Complexity avoidance**: Beads coordination seems more complex than worktrees

## Expected Failure Modes (Baseline)

Without multi-agent-coordination skill, agent likely:

1. **Uses git worktrees**: Creates separate worktrees for each agent
2. **Creates separate beads state**: Each worktree has divergent .beads/
3. **Plans manual merging**: "We'll merge the branches when done"
4. **Ignores bv --robot-plan**: Doesn't check which tasks can actually parallelize
5. **No atomic claiming**: Tasks aren't claimed, agents might pick same work

## Test Command

Run subagent with superpowers but WITHOUT multi-agent-coordination:

```
Task subagent with:
- Access to superpowers skills (except multi-agent-coordination)
- Working directory with .beads/ and multiple ready tasks
- User prompt: "I want to parallelize this - can we have multiple agents working at once?"

Document:
- Did agent suggest git worktrees or beads coordination?
- Did agent consider beads state divergence?
- Did agent use bv --robot-plan to check parallelization?
- Did agent suggest atomic task claiming?
```

## Success Criteria for Skill

After skill is written, same scenario should result in:
1. Agent recommends beads coordination over worktrees
2. Agent runs bv --robot-plan to identify parallelizable tasks
3. Agent explains atomic task claiming via bd update --status in_progress
4. Agent warns about when beads coordination doesn't work (different deps)
