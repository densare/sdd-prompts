# SDD Phase: VERIFY END-ISSUE

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.
> Run this AFTER `/sdd-end-issue` to verify everything was completed correctly.

## Role

Independent auditor that verifies all end-issue steps were executed correctly. Catches incomplete work (missing push, missing PR, open branch, wrong Linear state, etc.).

## When to Use

- After `/sdd-end-issue` executed by another agent (Kimi, Codex, etc.)
- When you suspect end-issue may have been incomplete
- As a routine quality check before moving on to the next issue

## Arguments

Format: `[ISSUE-ID]` (e.g., DT-48, DS-366 — a Linear issue identifier)

## Required Context

- `AGENTS.md` — for repository path, build/test commands
- Access to the code repository (git)
- Access to Linear (MCP or API)
- Access to GitHub (gh CLI)

## Linear Access

Issues are tracked in **Linear** (project management tool). To query/update issues:
- **MCP tools**: If available (e.g., `mcp__linear__get_issue`, `mcp__linear__update_issue`)
- **GraphQL API**: `https://api.linear.app/graphql`
- **CLI**: `linear-cli` if installed

## Verification Checklist

Run ALL checks below. For each, record PASS or FAIL with details.

### 1. Linear Issue State

```
QUERY Linear for [ISSUE-ID]
IF status != "Done" → FAIL: "Linear issue is [status], expected Done"
IF status == "Done" → PASS
```

### 2. Feature Branch — Must NOT Exist

The feature branch should have been deleted after merge.

```bash
# Check local branch
git branch --list "feature/[ISSUE-ID]" "*[ISSUE-ID]*"
# If found → FAIL: "Local branch still exists"

# Check remote branch
git ls-remote --heads origin "feature/[ISSUE-ID]" "*[ISSUE-ID]*"
# Also try the Linear-suggested branch name from the issue
# If found → FAIL: "Remote branch still exists"
```

**Note**: Branch name may not always be `feature/[ISSUE-ID]`. Also check:
- The `gitBranchName` field from the Linear issue
- Pattern `*[ISSUE-ID]*` to catch variations

### 3. Main Branch — Up to Date

```bash
git checkout main
git fetch origin
git status
# If behind origin/main → FAIL: "main is behind remote"
# If ahead of origin/main → FAIL: "main has unpushed local commits"
# If clean and up to date → PASS
```

### 4. Commit Exists on Main

```bash
git log --oneline --grep="[ISSUE-ID]" main
# If no commits found → FAIL: "No commit with [ISSUE-ID] in message found on main"
# If found → PASS, record commit hash and message
```

### 5. Push Verified

```bash
git log --oneline --grep="[ISSUE-ID]" origin/main
# If commit exists on origin/main → PASS
# If not → FAIL: "Commit exists locally but not on remote"
```

### 6. PR Exists and is Merged

```bash
gh pr list --search "[ISSUE-ID]" --state merged
# If found → PASS, record PR number and URL
# If not found, also try:
gh pr list --search "[ISSUE-ID]" --state all
# If found but open → FAIL: "PR exists but is not merged"
# If found but closed (not merged) → FAIL: "PR was closed without merging"
# If not found at all → FAIL: "No PR found for [ISSUE-ID]"
```

### 7. check-report.md Cleaned Up

```bash
ls check-report.md 2>/dev/null
# If exists → FAIL: "check-report.md was not deleted"
# If not exists → PASS
```

### 8. Working Directory Clean

```bash
git status --porcelain
# If empty → PASS
# If has changes → WARNING: "Uncommitted changes present" (list them)
```

### 9. End-Issue Report Files — Harvest and Delete

End-issue agents (Kimi, Codex, etc.) often leave report files in the repository root. These contain useful data but should not stay there.

**Find them:**
```bash
# Search for common patterns in the repo root
ls -1 end-issue-*.md sdd-end-issue-*.md *end-issue*[ISSUE-ID]*.md 2>/dev/null
```

**If found**, for EACH report file:

1. **READ** the file
2. **EXTRACT** the following into a structured note:
   - Commit hash
   - PR URL
   - Files changed (new + modified, with LOC if available)
   - Test results summary (pass/fail counts)
   - Any "Próximos Passos" / "Next Steps" / follow-up notes
   - Any deferred items or known issues
3. **APPEND** the extracted info to the `issues.md` in the planning repo (see Locate issues.md below)
4. **DELETE** the report file from the code repo

**Locate `issues.md`:** The planning repo has the task's issues.md at:
```
planning/aec/projectos/<project>/requests/em-implementacao/<task-folder>/issues.md
```
To find the right task folder:
- Query Linear for [ISSUE-ID] to get the title
- Search for [ISSUE-ID] in issues.md files under `planning/aec/projectos/`
- The issues.md that references [ISSUE-ID] is the target

**Append format** — add to the `## Historico` section at the bottom of issues.md:
```markdown
| <date> | [ISSUE-ID] Done — commit [HASH], PR [URL]. [summary of changes]. |
```

If `issues.md` has no `## Historico` section, create one at the bottom.

**If issues.md is NOT found** (e.g., task already archived), save the note to a temporary file:
```
planning/aec/projectos/<project>/requests/_notas-end-issue-[ISSUE-ID].md
```

**If NO report files found** → PASS (nothing to clean).

## Auto-Fix Rules

For each FAIL, attempt automatic repair:

| Check | Auto-Fix | Confirm First? |
|-------|----------|----------------|
| Linear not Done | `update_issue` → Done | No |
| Local branch exists | `git branch -d feature/[ISSUE-ID]` | No |
| Remote branch exists | `git push origin --delete feature/[ISSUE-ID]` | Yes |
| main behind remote | `git pull origin main` | No |
| Commit not on remote | `git push origin main` | **Yes** |
| PR not found | Create PR + merge | **Yes** |
| PR open not merged | `gh pr merge --merge` | **Yes** |
| check-report.md exists | `rm check-report.md` | No |
| End-issue reports found | Extract info → append to issues.md → delete file | No |

**"Confirm First? = Yes"** means ASK the user before executing. These are potentially destructive or externally visible actions.

**If commit is missing entirely** (not on main, not on any branch): **STOP**. This cannot be auto-fixed — the work may be lost. Inform the user immediately.

## Output

### All Passed

```
[ISSUE-ID] VERIFY-END: ALL PASSED ✓

  Linear:    Done
  Commit:    [HASH] on main
  Push:      Verified on origin/main
  PR:        [URL] (merged)
  Branch:    Cleaned up (local + remote)
  Cleanup:   check-report.md removed
  Reports:   [N] harvested → issues.md updated, files deleted
  Workspace: Clean
```

### Failures Found

```
[ISSUE-ID] VERIFY-END: [N] ISSUES FOUND

  Check              Status    Detail
  ─────              ──────    ──────
  Linear state       PASS      Done
  Feature branch     FAIL      Remote branch still exists
  main up to date    PASS      —
  Commit on main     PASS      abc1234
  Push verified      PASS      —
  PR merged          PASS      #42 https://github.com/...
  check-report.md    PASS      Removed
  Workspace          WARNING   2 uncommitted files

  Auto-fixed:
  - Deleted remote branch feature/DT-48

  Remaining:
  - (none, or list what could not be fixed)
```

## Rules

- This is a READ + VERIFY command — only modify state to fix detected issues
- Never force push
- Never delete main or other non-feature branches
- If auto-fix fails, report the failure and stop — do not retry
- If the issue ID is not found in Linear, STOP immediately
