# SDD Phase: BUG (Investigate and Plan Fix)

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Act as a **Bug Investigator**. Find the root cause of a bug, document it, and plan the fix. Do NOT implement the fix — only investigate and plan.

## Required Context

Read or have access to:
- `sdd/ANTI_PATTERNS.md` — **MANDATORY** anti-patterns to avoid in the fix plan
- `AGENTS.md` — project rules, conventions, build/test commands, layer architecture

## Arguments

Two modes:

| Input | Mode | Example |
|-------|------|---------|
| `[ISSUE-ID]` | **Existing bug issue**: read from Linear, investigate, plan fix | `/sdd-bug DT-48` |
| `"free text"` | **New bug**: investigate, plan fix, create issue if needed | `/sdd-bug "Nic calculation returns zero"` |

Detection: if it starts with a known project prefix (DT-, DS-, SM-) it's an existing issue. Otherwise it's a free text description.

## Linear Access

Issues are tracked in **Linear** (project management tool). To query/create issues:
- **MCP tools**: If available (e.g., `mcp__linear__get_issue`, `mcp__linear__create_issue`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

## Behavior

### Step 0: Detect Context

```bash
git branch --show-current
```

Determine working context:
- **Existing issue mode** (`[ISSUE-ID]` provided): Query Linear for the issue. This is a known bug that needs investigation.
- **Free text mode on a feature branch** (`feature/XX-NNN`): We are in active implementation. The bug is related to the current work. Extract the issue ID from the branch name.
- **Free text mode on `main`**: No active implementation. This is a standalone bug.

### Step 1: Confirm the Bug

1. **REPRODUCE**: Try to reproduce the problem
   - Run the project's build command (see AGENTS.md)
   - Run the project's test command (see AGENTS.md)
   - If specific reproduction steps are known, follow them
2. **VERIFY**: Is the bug real? Not a false positive?
3. If cannot confirm: inform that the bug cannot be reproduced. STOP.

### Step 2: Find Root Cause

1. **LOCATE** relevant code (Grep/Glob)
2. **READ** the files involved
3. **IDENTIFY** root cause:
   - Wrong logic? Incorrect formula? Silent exception?
   - Regression from another issue? Code never implemented?
   - Missing validation? Wrong layer? (AP-06)
4. **REFERENCE** the original spec or issue where the behavior was defined:
   - Search Linear for related Done issues
   - Search planning repository for related specs

### Step 3: Plan the Fix

1. **ASSESS** complexity:
   - **Simple** (1-2 files, obvious fix): plan directly
   - **Complex** (3+ files, redesign needed): plan in detail
2. **LIST** files to change with description of each change
3. **VERIFY** the fix plan against anti-patterns:
   - Fix doesn't introduce AP-01 to AP-08?
   - Fix in the correct layer?
   - Fix within LOC limits?
4. **ESTIMATE** in Fibonacci points (1-5 typical for bugs)

### Step 4: Create/Update Linear Issue

**Existing issue mode**:
- Add a comment to the issue with the investigation results

**Free text mode on feature branch** (`feature/XX-NNN`):
- Add a comment to the issue XX-NNN with the investigation results
- Do NOT create a new issue — the fix is part of the current work

**Free text mode on `main`**:
- Create a NEW Linear issue:
  ```
  Title: "Bug: <short description>"
  Priority: based on impact (1=Urgent, 2=High, 3=Normal, 4=Low)
  Labels: bug
  ```

### Step 5: Write bug-report.md

Write `bug-report.md` in the **root of the code repository** with the investigation results:

```markdown
## Bug Report: [ISSUE-ID or description]

> **Context**: Existing issue | Active branch (feature/XX-NNN) | New bug (main)
> **Date**: [date]

### Bug Description
[What's wrong — observed vs expected behavior]

### Reproduction
- Reproduced: Yes / No
- Steps: [how to reproduce]
- Build: PASS / FAIL
- Tests: X/Y passing

### Root Cause

**Cause**: [technical explanation]
**File(s)**: `path:line`
**Original spec/issue**: [reference where behavior was defined]

### Fix Plan

| File | Change | Layer |
|------|--------|-------|
| `path` | [what to change] | Domain/Application/Infrastructure/Presentation |

### Steps
1. [step 1]
2. [step 2]

### Anti-Pattern Check
- [ ] Fix doesn't introduce AP-01 to AP-08
- [ ] Fix in correct layer (AP-06)
- [ ] Fix within LOC limits (AP-05)

### Estimate
[X] points

### Verification
- [ ] [how to confirm the fix works]
- [ ] Build passes
- [ ] Tests pass

### Linear
- **Issue**: [XX-NNN](url) — [created / updated / commented]
```

> **Note**: This file should NOT be committed to git. The implementer reads it, applies the fix, then deletes it.

### Step 6: Inform

Report to the user:
- Issue ID and URL
- Root cause (summary)
- Fix plan (summary)
- Next step: `/sdd-implement [ISSUE-ID]` (if new issue or existing issue) or `/sdd-fix [ISSUE-ID]` (if fixing within active branch)

## Rules

- Do NOT fix the bug — only investigate and plan
- If the bug is in a different module: inform and suggest creating the issue in the correct team
- If root cause is ambiguous: document hypotheses and suggest further investigation
- If fix is trivial (1 line): include the suggested code in the report
- If on a feature branch: the fix is part of the current work, do NOT create a new issue

## Output

`bug-report.md` written to the code repo root. Linear issue created/updated. Ready for implementation.
