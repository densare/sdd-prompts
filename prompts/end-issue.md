# SDD Phase: END ISSUE

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Finalize the issue: final validation, commit, **PUSH**, **PR**, merge, close in Linear.

## CRITICAL: Complete Workflow is Mandatory

This command MUST execute **ALL** steps below — from validation through merge and cleanup.
**NEVER** stop at just the local commit. A commit without push+PR+merge is an incomplete end-issue.

## Required Context

- Code implemented and verified (IMPLEMENT + CHECK phases)
- Currently on branch `feature/[ISSUE-ID]`
- `AGENTS.md` — for build/test/format commands

## Arguments

Format: `[ISSUE-ID]` (e.g., DT-48, DS-366 — a Linear issue identifier)

## Linear Access

Issues are tracked in **Linear** (project management tool). To query/update issues:
- **MCP tools**: If available (e.g., `mcp__linear__get_issue`, `mcp__linear__update_issue`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

## Locate Issue

1. **QUERY LINEAR** for `[ISSUE-ID]` to get: title, description, status, branch name
2. Verify the issue is not already Done

## Pre-condition: Approved Review

Before proceeding, verify that `check-report.md` exists in the **root of the code repository** with an approved verdict.

```
IF check-report.md does not exist:
  → STOP: "No review report found. Run /sdd-check first."

IF check-report.md verdict is REJECTED:
  → STOP: "Review report is REJECTED. Run /sdd-fix first, then /sdd-check again."

IF check-report.md verdict is APPROVED or APPROVED WITH DEFERRALS:
  → IF APPROVED WITH DEFERRALS: verify that EVERY deferral in the Deferrals table has a Linear issue ID.
    If any deferral is missing a Linear issue → STOP: "Deferrals must have Linear issues before merge."
  → Continue with end-issue steps.
```

## Steps

### 1. Final validation with rebase

```bash
git fetch origin && git rebase origin/main
```

If conflicts: resolve them.

Check if rebase brought structural changes (project/build configuration files):
- If YES: clean rebuild (see AGENTS.md for build commands)
- If NO but rebase brought changes: incremental build
- If rebase brought no changes: skip build

In any case, run:
- **Tests** (see AGENTS.md for test command)
- **Format check** (see AGENTS.md for format command, if applicable)

Fix errors until everything passes.

### 2. Stage files

Stage only files relevant to the issue (DO NOT use `git add -A`):
```bash
git add src/path/to/changed/files
git add tests/path/to/test/files
```

**CRITICAL — No files left behind:** After staging, run `git status` and check for unstaged/untracked files. If ANY exist, **ASK the user** whether they should be included in the commit or intentionally excluded. NEVER silently leave files behind.

### 3. Commit

```bash
git commit -m "[ISSUE-ID]: description of what was implemented"
```

### 4. Push — MANDATORY

**NEVER omit this step.**

```bash
git push -u origin feature/[ISSUE-ID]
```

Verify push was successful:
```bash
git log origin/feature/[ISSUE-ID]..HEAD  # must be empty (no unpushed commits)
```

### 5. Create PR — MANDATORY

**NEVER omit this step.**

```bash
gh pr create --title "[ISSUE-ID]: descriptive title" --body "## Summary
- point 1
- point 2

## Test plan
- [ ] Build passes
- [ ] Tests pass
- [ ] Manual verification (if applicable)
"
```

Save the returned PR URL — it is required in the output.

### 6. Merge

```bash
gh pr merge --merge --admin
```

### 7. Close issue in Linear

Update issue [ISSUE-ID] in Linear to state "Done".

### 8. Update STATUS.md in planning repo (if exists)

After closing the issue in Linear, update the planning repo's STATUS.md to reflect the completed issue.

**How to locate STATUS.md:**
1. Check `AGENTS.md` (or project configuration) for the planning repo path
2. From the issue ID prefix, determine the project folder:
   - `PLT-*` → `plataforma-base`, `DT-*` → `dentherm`, `DS-*` → `denstudio`, etc.
3. Look for `STATUS.md` at `projectos/<project>/STATUS.md` in the planning repo

**If STATUS.md exists:**
1. Update the issue's row in the progress table (e.g., change `NOT STARTED` → `Done` for code status)
2. Update the "Last update" timestamp
3. If ALL issues for a request are now Done, update the request's overall state

**If STATUS.md does not exist or planning repo is not accessible:** skip this step silently.

### 9. Cleanup — delete check-report and feature branch

After merge, clean up:
```bash
# Remove review report (transient artifact, not committed)
rm -f check-report.md

# Delete feature branch (local and remote)
git checkout main && git pull origin main
git branch -d feature/[ISSUE-ID]
git push origin --delete feature/[ISSUE-ID]
```

## Rules — NEVER DO

- **NEVER** say "Ready for merge" or "Ready for PR" — that implies unfinished work
- **NEVER** omit the push (step 4)
- **NEVER** omit PR creation (step 5)
- **NEVER** omit the PR URL from the output
- **NEVER** force push (`--force`)
- **NEVER** use `git add -A` (may include unintended files)
- If any step fails: **STOP** and inform the user
- If tests fail: fix BEFORE continuing

## Exit Checklist — Verify Before Responding

Before reporting completion to the user, confirm ALL of these:

- [ ] Commit exists with hash
- [ ] **Push was executed** (code is on the remote)
- [ ] **PR was created** and URL was obtained
- [ ] PR was **merged**
- [ ] Issue is **Done** in Linear
- [ ] Feature branch cleaned up
- [ ] `check-report.md` removed
- [ ] STATUS.md updated (if planning repo accessible)
- [ ] Report `end-issue-[ISSUE-ID].md` includes the **PR URL**

If ANY item is unchecked, do NOT report completion — go back and complete it.

## Output

Report to the user in this format:

```
[ISSUE-ID] END-ISSUE COMPLETED

  Commit:  [HASH]
  Branch:  feature/[ISSUE-ID]
  PR:      [CLICKABLE URL]
  Linear:  Done

Report: end-issue-[ISSUE-ID].md
```

The **PR URL** in the output is MANDATORY. If missing, the end-issue is incomplete.
