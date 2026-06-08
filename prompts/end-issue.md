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

**Use `scripts/linear.py` for ALL Linear ops** — read, list, create, update, comment, label:

```bash
python "$ORCH_HOME/scripts/linear.py" get DT-NNN                                          # full issue (description, labels, parent/children, relations)
python "$ORCH_HOME/scripts/linear.py" update DT-NNN --state Done --comment "Merged via PR #1234"
python "$ORCH_HOME/scripts/linear.py" label DT-NNN --add tech-debt
python "$ORCH_HOME/scripts/linear.py" create --team DenTherm --title "..." --description "..." --priority 3 --labels tech-debt
```

`$ORCH_HOME` is exported by the orchestrator; API key auto-discovered from `<sandbox>/.linear.env`.

**DO NOT use Linear MCP** (`mcp__linear__*`). The MCP was detached from sandbox `.mcp.json` on 2026-06-01 — its tool schemas + verbose responses were consuming ~50% of session context. Calling `mcp__linear__*` will fail. If linear.py is missing an op, extend it — don't fall back to MCP or curl.

## Deferral Hygiene

If you open a Linear deferral during this phase (rare for end-issue but allowed — e.g., when verifying APPROVED WITH DEFERRALS surfaces additional items), the new issue **MUST** carry the `tech-debt` label **plus every label its source issue carries** (a deferral of an `F1` issue is `F1`; an epic child inherits the **epic's** label — read them with `linear.py get <SOURCE-ISSUE>`). See `SDD_DISCIPLINE.md` §Rule 3 for rationale. One-liner via the helper script (auto-creates the label if missing):

```bash
python "$ORCH_HOME/scripts/linear.py" create \
  --team DenTherm --title "<deferral title>" \
  --description "<context + link to parent + scope>" \
  --priority 3 --estimate 1 --labels "tech-debt,<each label the source issue/epic carries>"
```

When **verifying** existing deferrals from `check-report.md`: if any deferral Linear issue is missing the `tech-debt` label **or any label its source carries**, apply them now (idempotent):
```bash
python "$ORCH_HOME/scripts/linear.py" label DT-NNN --add "tech-debt,<source labels>"
```
Do not block merge over a missing label — fix it and proceed.

An unlabelled deferral is a defect — and a deferral that drops its source's scope labels (e.g. `F1`) silently escapes that scope. Carry `tech-debt` **plus** the source's labels so operators' scope/tech-debt filters both stay correct.

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
  → Continue to "Pre-condition: Smoke Tests Confirmed" below.
```

## Pre-condition: Smoke Tests Confirmed

Manual smoke tests are operator-driven (the AI agent cannot drive a UI / runtime). Even if `/sdd-check` documented smokes as passed, the operator must **explicitly re-confirm** before merge — smoke results can change between check time and end-issue time (e.g., rebase brought new changes, environment drift).

**ASK the operator EXPLICITLY** (do not assume; do not skip even if `check-report.md` says they passed):

> *"All manual smoke tests from `plan.md` §Planned tests passed without errors? Reply with: each smoke # + PASS/FAIL + observations. If `plan.md` has no smoke list, confirm that no observable acceptance criterion requires manual verification."*

```
IF operator answers anything other than unambiguous PASS for every smoke:
  → STOP: "Smoke tests not confirmed (or failures reported). Fix failures, update plan.md if scope changed, and re-run /sdd-check before /sdd-end-issue."

IF plan.md has no manual smoke tests AND the issue body has observable acceptance criteria
   (e.g., "lista mostra X", "painel abre Y", "valor persiste após reload"):
  → STOP: "Plan deficiency — observable acceptance criteria require enumerated manual smoke tests.
            Update plan.md, run smokes, then re-run /sdd-check before /sdd-end-issue."

IF operator confirms PASS for all smokes (or plan has no smokes AND no observable criteria):
  → Continue with end-issue steps.
```

This gate is **mandatory** and cannot be skipped. A merge without operator smoke confirmation is treated as an incomplete end-issue.

## Steps

### 1. Final validation with rebase

```bash
git fetch origin && git rebase origin/main
```

If conflicts: resolve them.

The CHECK phase already verified build + the FULL test suite green on this branch. Re-verify only as much as the rebase actually warrants:
- **Rebase brought NO changes** (branch already current — the common case): the check's green still holds. Do a **build-only** sanity check (see AGENTS.md); **do NOT re-run the full test suite** — it is redundant here and, on a large solution, the single slowest step.
- **Rebase brought structural changes** (project/build configuration files): clean rebuild + run the **full test suite**.
- **Rebase brought non-structural changes**: incremental build + run the **full test suite**.
- **Format check** (see AGENTS.md for the format command, if applicable).

**CRITICAL — build/test are FOREGROUND, BLOCKING commands.** Run each one and **wait for its exit code within THIS turn**, then read the result and proceed. A headless run has **no notification, callback, or "build finished" event to wait for**: NEVER background the build/test (`&`, "run in background", "let me wait for the notification/build to finish") and end your turn — that aborts the end-issue mid-flight, leaving the branch un-pushed and the phase reported `UNKNOWN` (no merge). Block until the command returns its exit code.

Fix any errors until build + tests pass, then continue.

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
- **NEVER** background the build/test or "wait for a notification/callback" for them to finish — they are foreground, blocking commands; block until they return in this turn. Headless runs have no async callback, so backgrounding them and ending the turn aborts the merge (phase reported `UNKNOWN`, branch un-pushed).
- If any step fails: **STOP** and inform the user
- If tests fail: fix BEFORE continuing

## Exit Checklist — Verify Before Responding

Before reporting completion to the user, confirm ALL of these:

- [ ] **Manual smoke tests confirmed PASS by operator** (Pre-condition: Smoke Tests Confirmed)
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
