# SDD Phase: FIX (Apply Review Corrections)

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Act as an **Implementer** applying corrections from a code review. Read the review report, fix every issue, then signal readiness for re-review.

## Mandatory Reading — BEFORE ANY CODE

**STOP. Read these files FIRST, before writing a single line of code.**

You MUST actually open and read each file below. "Having access to" is NOT enough — you must read the content into your context. Failure to read these files leads to repeated review rejections and wrong fixes.

1. **`sdd/ANTI_PATTERNS.md`** — Anti-patterns to avoid (AP-01 through AP-08). Contains LOC limits, abstraction rules, and self-check questions.
2. **`sdd/QUALITY_GATES.md`** — Mandatory checks (section Gate 3: IMPLEMENT). Contains the quality criteria your fix will be re-reviewed against.
3. **`AGENTS.md`** — Project rules, conventions, build/test commands.

**Do NOT proceed to the Behavior section until you have read all three files.**

## Arguments

Format: `[ISSUE-ID]` (e.g., DT-48, DS-366 — a Linear issue identifier)

## Linear Access

Issues are tracked in **Linear** (project management tool). To query issues:
- **MCP tools**: If available (e.g., `mcp__linear__get_issue`, `mcp__linear__list_issues`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

## Locate Review Report

1. **READ** `check-report.md` from the **root of the current code repository**
2. If `check-report.md` does not exist: STOP — no review to fix. Run `/sdd-check` first.
3. If `check-report.md` has verdict **APPROVED**: STOP — nothing to fix.
4. **QUERY LINEAR** for `[ISSUE-ID]` to get: title, description, status
5. **FIND** the corresponding request folder in the planning repository (search `projectos/*/requests/em-implementacao/*/issues.md` for the issue ID)
6. **READ** `spec.md` and `plan.md` from that folder for context

## Pre-conditions

- `check-report.md` exists with verdict **REJECTED**
- Currently on branch `feature/[ISSUE-ID]`
- Code compiles (even if with issues flagged in the review)

## Behavior

1. **READ MANDATORY FILES** — If you haven't already, read `sdd/ANTI_PATTERNS.md`, `sdd/QUALITY_GATES.md`, and `AGENTS.md` now. Do NOT skip this.
2. **READ** `check-report.md` completely
3. **LIST** all FAIL items from the report tables:
   - Requirements Verification (RF-XX with FAIL)
   - Anti-Pattern Verification (AP-XX with FAIL)
   - Build and Tests (any FAIL)
   - Corrective actions listed at the bottom
4. **FOR EACH** FAIL item:
   - Understand what the reviewer flagged
   - Locate the relevant code
   - Apply the correction following the same rules as `/sdd-implement`:
     - RED/GREEN/VERIFY for requirement fixes (write test, fix code, verify)
     - Respect LOC limits, layer architecture, anti-patterns (from the files you read in step 1)
     - MODIFY existing files, don't create new ones unless the review explicitly asks for split
5. **VERIFY** after all corrections:
   - Run build command (zero errors, zero warnings)
   - Run test command (zero failures)
   - Run format check command
   - Confirm each FAIL item is now addressed
6. **DELETE** `check-report.md` from the code repository root
   - This signals to the reviewer: "corrections applied, ready for re-review"
7. **COMMIT** the fixes:
   ```bash
   git add <fixed-files>
   git commit -m "[ISSUE-ID]: fix review findings"
   git push
   ```
8. **INFORM** the user that corrections are applied and the code is ready for `/sdd-check`

## Out of Scope — NEVER DO in this phase

- **NEVER** mark the Linear issue as Done or update its status (that is END-ISSUE step 7)
- **NEVER** create PRs or merge (that is END-ISSUE)
- **NEVER** close or comment on GitHub Issues
- **NEVER** run `/sdd-check` or `/sdd-end-issue` — those are separate phases

This phase ONLY fixes review findings and pushes. The workflow continues with CHECK re-review.

## Rules

- ONLY fix what the review flagged — don't add unrelated changes
- DO NOT refactor beyond what the review asks
- If a review finding is unclear or you disagree: STOP and ask the user
- If a fix requires changes outside the scope of the issue: STOP and inform
- **ZERO TOLERANCE for build errors and test failures** after fixes
- After deleting `check-report.md`, do NOT run `/sdd-check` yourself — the reviewer (different model/session) must do it

## Output

All FAIL items from the review corrected. `check-report.md` deleted. Code pushed. Ready for `/sdd-check` re-review.
