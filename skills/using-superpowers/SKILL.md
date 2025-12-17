---
name: using-superpowers
description: Use when starting any conversation - establishes mandatory workflows for finding and using skills, including using Skill tool before announcing usage, following brainstorming before coding, and creating TodoWrite todos for checklists
---

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST read the skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

# Getting Started with Skills

## MANDATORY FIRST RESPONSE PROTOCOL

Before responding to ANY user message, you MUST complete this checklist:

0. ☐ **If project has `.beads/`**: Run `bd ready --json` and `bd list --status in_progress --json` FIRST
   - In-progress work? → Use **session-start-with-beads** skill to continue
   - Ready work but nothing in-progress? → Present options, let user choose
   - Beads determines WHAT to work on, THEN check skills for HOW

1. ☐ List available skills in your mind
2. ☐ Ask yourself: "Does ANY skill match this request?"
3. ☐ If yes → Use the Skill tool to read and run the skill file
4. ☐ Announce which skill you're using
5. ☐ Follow the skill exactly

**Responding WITHOUT completing this checklist = automatic failure.**

## Critical Rules

1. **Follow mandatory workflows.** Brainstorming before coding. Check for relevant skills before ANY task.

2. Execute skills with the Skill tool

## Common Rationalizations That Mean You're About To Fail

If you catch yourself thinking ANY of these thoughts, STOP. You are rationalizing. Check for and use the skill.

- "This is just a simple question" → WRONG. Questions are tasks. Check for skills.
- "I can check git/files quickly" → WRONG. Files don't have conversation context. Check for skills.
- "Let me gather information first" → WRONG. Skills tell you HOW to gather information. Check for skills.
- "This doesn't need a formal skill" → WRONG. If a skill exists for it, use it.
- "I remember this skill" → WRONG. Skills evolve. Run the current version.
- "This doesn't count as a task" → WRONG. If you're taking action, it's a task. Check for skills.
- "The skill is overkill for this" → WRONG. Skills exist because simple things become complex. Use it.
- "I'll just do this one thing first" → WRONG. Check for skills BEFORE doing anything.
- "I'll check beads after I understand the request" → WRONG. Beads first. Context determines response.
- "Project might not use beads" → Check for `.beads/` directory. 2 seconds.

**Why:** Skills document proven techniques that save time and prevent mistakes. Not using available skills means repeating solved problems and making known errors.

If a skill for your task exists, you must use it or you will fail at your task.

## Skills with Checklists

If a skill has a checklist, YOU MUST create TodoWrite todos for EACH item.

**Don't:**
- Work through checklist mentally
- Skip creating todos "to save time"
- Batch multiple items into one todo
- Mark complete without doing them

**Why:** Checklists without TodoWrite tracking = steps get skipped. Every time. The overhead of TodoWrite is tiny compared to the cost of missing steps.

## TodoWrite vs Beads

Both coexist at different timescales:

| Tool | Scope | Lifespan | Use For |
|------|-------|----------|---------|
| **TodoWrite** | Current session | Ephemeral (gone after session) | Execution steps, checklist tracking |
| **Beads (bd)** | Cross-session | Persistent (survives compaction) | Strategic work items, outcomes |

**Pattern:**
1. Beads tracks the work item (e.g., "Implement feature X")
2. TodoWrite tracks execution steps within this session
3. Update beads notes at milestones (70% tokens, task complete)
4. Close beads when outcome achieved

Don't duplicate tracking. TodoWrite = fine-grained execution. Beads = outcome-focused.

For detailed bd/bv commands: `~/.claude/skills/beads/` and `~/.claude/skills/using-bv/`.

## Announcing Skill Usage

Before using a skill, announce that you are using it.
"I'm using [Skill Name] to [what you're doing]."

**Examples:**
- "I'm using the brainstorming skill to refine your idea into a design."
- "I'm using the test-driven-development skill to implement this feature."

**Why:** Transparency helps your human partner understand your process and catch errors early. It also confirms you actually read the skill.

# About these skills

**Many skills contain rigid rules (TDD, debugging, verification).** Follow them exactly. Don't adapt away the discipline.

**Some skills are flexible patterns (architecture, naming).** Adapt core principles to your context.

The skill itself tells you which type it is.

## Instructions ≠ Permission to Skip Workflows

Your human partner's specific instructions describe WHAT to do, not HOW.

"Add X", "Fix Y" = the goal, NOT permission to skip brainstorming, TDD, or RED-GREEN-REFACTOR.

**Red flags:** "Instruction was specific" • "Seems simple" • "Workflow is overkill"

**Why:** Specific instructions mean clear requirements, which is when workflows matter MOST. Skipping process on "simple" tasks is how simple tasks become complex problems.

## Summary

**Starting any task:**
1. If `.beads/` exists → Check bd state first (WHAT to work on)
2. If relevant skill exists → Use the skill (HOW to do it)
3. Announce you're using it
4. Follow what it says

**Skill has checklist?** TodoWrite for every item.

**Finding a relevant skill = mandatory to read and use it. Not optional.**
