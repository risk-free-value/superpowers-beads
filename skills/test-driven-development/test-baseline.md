# TDD Delete vs Discard - Test Baseline

## Problem Statement

CLAUDE.md rule: "You may NOT delete any file or directory unless I explicitly give the exact command"

TDD skill phrases that APPEAR to conflict:
- "Delete it. Start over." (line 37)
- "Delete means delete" (line 43)
- "Delete means delete" in rationalization table (line 265)
- "Delete code. Start over with TDD." (line 288)

## The Actual Intent

TDD skill is about **discarding uncommitted work**, not deleting files:
- You wrote implementation code before tests
- You need to revert that work to start fresh
- The goal: don't let pre-test code influence test design

## Why Confusion Occurs

"Delete" could mean:
1. `rm file.ts` - actually removing files (what CLAUDE.md prohibits)
2. `git checkout file.ts` - reverting changes (what TDD means)
3. `git stash` - saving changes aside for reference

Without clarification, an agent might:
- Refuse to follow TDD (interpreting as CLAUDE.md violation)
- Actually run `rm` commands (wrong action)
- Be confused about what action to take

## Required Changes

1. Change "Delete it" → "Discard it"
2. Change "Delete means delete" → "Discard means discard"
3. Add clarification box explaining:
   - "Discard" means `git checkout` or `git stash`
   - NOT `rm` to delete files
   - Principle: start fresh so tests drive design
4. Update red flags consistently

## Acceptance Criteria

- [ ] "Delete" changed to "discard" throughout (where referring to work/code)
- [ ] Clarification box added explaining git checkout/stash
- [ ] No references to rm or file deletion
- [ ] Spirit of TDD preserved (don't keep pre-test code)
