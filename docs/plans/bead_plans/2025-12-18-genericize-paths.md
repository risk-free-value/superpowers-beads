# Plan 3: Genericize Example Paths

## Executive Summary

The original proposal (`/Users/jesse/` → `/Users/dev/`) doesn't meaningfully improve clarity. After first-principles analysis, use **`/home/user/`** as the standard placeholder - it's universally understood as generic in documentation.

## Problem Statement

Some skills contain example file paths referencing the original author's home directory (`/Users/jesse/...`). These should be genericized for clarity and universality.

## Scope Verification (Actual Findings)

Investigation found **4 occurrences** of `/Users/jesse/`:

| File | Line | Context | Action |
|------|------|---------|--------|
| `skills/root-cause-tracing/SKILL.md` | 41 | Example error message | **CHANGE** |
| `skills/using-git-worktrees/SKILL.md` | 189 | Example workflow output | **CHANGE** |
| `skills/systematic-debugging/CREATION-LOG.md` | 7 | Historical attribution | **KEEP** |
| `docs/plans/writing-skills-cleanup-step.md` | multiple | Meta-reference to this task | **N/A** |

**Important:** The CREATION-LOG.md instance is historical attribution documenting actual source ("Extracted debugging framework from `/Users/jesse/.claude/CLAUDE.md`"). This is factual metadata, not an example path. Changing it would be revisionist.

---

## First-Principles Analysis: Placeholder Convention

| Convention | Pros | Cons |
|------------|------|------|
| `/Users/dev/` | Looks like real path | Mac-specific, arbitrary name |
| `$PROJECT_ROOT/...` | Clear variable syntax | Agent might try to expand env var |
| `<project>/...` | Clearly a placeholder | Doesn't look like real error output |
| `/path/to/project/...` | Standard docs convention | Verbose |
| `/home/user/` | Universally generic "user" | Platform-neutral enough |

### Decision: Use `/home/user/`

**Rationale:**
1. `user` is universally understood as a placeholder username in documentation
2. `/home/user/` is platform-neutral enough (Mac users understand it)
3. Maintains realistic path structure for pattern recognition
4. Doesn't look like it belongs to any specific author
5. Consistent with common documentation practices

---

## Edit Specifications

### Edit 1: `skills/root-cause-tracing/SKILL.md` (line 41)

**Current:**
```
Error: git init failed in /Users/jesse/project/packages/core
```

**Change to:**
```
Error: git init failed in /home/user/project/packages/core
```

### Edit 2: `skills/using-git-worktrees/SKILL.md` (line 189)

**Current:**
```
Worktree ready at /Users/jesse/myproject/.worktrees/auth
```

**Change to:**
```
Worktree ready at /home/user/myproject/.worktrees/auth
```

### No Edit: `skills/systematic-debugging/CREATION-LOG.md` (line 7)

**Keep as-is:**
```
Extracted debugging framework from `/Users/jesse/.claude/CLAUDE.md`
```

**Rationale:** Historical attribution documenting actual source, not an example path.

---

## Verification Steps

After edits:

```bash
# Should only find CREATION-LOG.md and plan documents
grep -r '/Users/jesse/' skills/ --include='*.md' | grep -v 'CREATION-LOG'
```

---

## Acceptance Criteria

- [ ] `skills/root-cause-tracing/SKILL.md` line 41 updated to `/home/user/`
- [ ] `skills/using-git-worktrees/SKILL.md` line 189 updated to `/home/user/`
- [ ] `skills/systematic-debugging/CREATION-LOG.md` unchanged (historical attribution)
- [ ] No other `/Users/jesse/` paths remain in non-attribution contexts
- [ ] Consistent placeholder convention across all skills

---

## Note on using-git-worktrees

Since `using-git-worktrees` is marked for deletion in Plan 2, this edit may become moot. However, the edit should be done if:
- The skill is NOT deleted (Plan 2 not executed)
- OR the edit is done before Plan 2 execution
- OR we want git history to show the correction before deletion
