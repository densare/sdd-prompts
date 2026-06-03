# SDD Phase: IMPLEMENT

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Act as an **Implementer**. Implement code strictly following the approved plan.

## Mandatory Reading — BEFORE ANY CODE

**STOP. Read these files FIRST, before writing a single line of code.**

You MUST actually open and read each file below. "Having access to" is NOT enough — you must read the content into your context. Failure to read these files leads to anti-pattern violations, wrong LOC limits, and rejected reviews.

1. **`sdd/ANTI_PATTERNS.md`** — Anti-patterns to avoid (AP-01 through AP-08). Contains LOC limits, abstraction rules, and self-check questions you MUST apply throughout implementation.
2. **`sdd/QUALITY_GATES.md`** — Mandatory checks (section Gate 3: IMPLEMENT). Contains the quality criteria your code will be reviewed against.
3. **`AGENTS.md`** — Project rules, conventions, security rules, build/test commands, layer architecture.

**Do NOT proceed to the Behavior section until you have read all three files.**

## Arguments

Format: `[ISSUE-ID]` (e.g., DT-48, DS-366 — a Linear issue identifier). Implementation is always per-issue; the **mode** (below) only changes *where* the plan and spec live.

## Modes

`/sdd-implement` mirrors the two **mutually-exclusive modes** of `/sdd-plan` (canonical: `prompts/plan.md`). Detect the mode from **where `/sdd-plan` wrote the plan**:

### Mode A — Request planning (plan lives in planning repo)

- **Plan path:** `planning/aec/projectos/<project>/requests/em-implementacao/<request-id>/plan.md`.
- **Spec:** `spec.md` exists **in the same folder** and is REQUIRED.
- Located via the `issues.md` that lists `[ISSUE-ID]` (see §Locate).

### Mode B — Issue planning (plan lives in code repo)

- **Plan path:** `<code-repo>/docs/issues/<ISSUE-ID>/plan.md`.
- **Spec:** there is **no `spec.md`** and none is required — the **Linear issue body is the spec** (its "Acceptance criteria" / "Scope" sections are the requirements list). Do **NOT** search the planning repo in this mode.
- **Trigger:** the issue was created via `/sdd-check` deferral, UX review, bug report, or follow-up.

### Centralised record

This file (`prompts/implement.md` in `densare/sdd-prompts/master`) is the **canonical source** for the mode split. Each consumer repo links to it from `AGENTS.md §SDD` via the raw URL — propagation is automatic, no per-repo deploy.

## Linear Access

Issues are tracked in **Linear**. **Use `scripts/linear.py` for ALL Linear ops** — read, list, create, update, comment, label:

```bash
python "$ORCH_HOME/scripts/linear.py" get DT-NNN
python "$ORCH_HOME/scripts/linear.py" list --team DenTherm --open-only --limit 50 [--json]
python "$ORCH_HOME/scripts/linear.py" create --team DenTherm --title "..." --description "..." --priority 1 --labels tech-debt
python "$ORCH_HOME/scripts/linear.py" update DT-NNN --state "In Progress" --comment "..."
python "$ORCH_HOME/scripts/linear.py" label DT-NNN --add tech-debt
```

`$ORCH_HOME` is exported by the orchestrator; API key auto-discovered from `<sandbox>/.linear.env`.

**DO NOT use Linear MCP** (`mcp__linear__*`). The MCP was detached from sandbox `.mcp.json` on 2026-06-01 — its tool schemas + verbose responses were consuming ~50% of session context. Calling `mcp__linear__*` will fail.

## Deferral Hygiene

If you open a Linear deferral during this phase, the new issue **MUST** carry the `tech-debt` label in its team's workspace (see `SDD_DISCIPLINE.md` §Rule 3 for the full rule and rationale):

**Use the orchestrator helper** `scripts/linear.py` (direct GraphQL, **zero context bloat**) — DO NOT use Linear MCP for batch create/update/label (each MCP call inflates session context):

```bash
python "$ORCH_HOME/scripts/linear.py" create   --team DenTherm --title "<deferral title>"   --description "<why + scope + parent issue link>"   --priority 3 --labels tech-debt
```

The script auto-creates the `tech-debt` label if missing (color #6e6e6e, description per SDD_DISCIPLINE.md §Rule 3) and applies it during creation, then prints the new ID.

An unlabelled deferral is a defect — operators rely on the label to separate the tech-debt backlog from feature work.

## Locate Issue, Spec and Plan

1. **QUERY LINEAR** for `[ISSUE-ID]` to get: title, description (**acceptance criteria!**), status, labels, and any linked information.
2. **LOCATE THE PLAN — try Mode B first, then Mode A:**
   - **Mode B (code repo):** look for `docs/issues/<ISSUE-ID>/plan.md` in the current code repo. If it exists → **Mode B**. There is **no `spec.md`**; the Linear issue body (step 1) is the spec. Do not search the planning repo.
   - **Mode A (planning repo):** otherwise, find the `issues.md` that references `[ISSUE-ID]` (search `projectos/*/requests/em-implementacao/*/issues.md`). From that folder, read `spec.md` **and** `plan.md`.
3. **READ** the located `plan.md` in full (Mode A: also read `spec.md` in full).
4. If neither location yields a `plan.md`, inform the user and **STOP** — do not improvise a plan (`/sdd-plan` must run first).

## Pre-conditions

- Linear issue exists and is not Done/Cancelled
- `plan.md` exists and is approved (PLANNED)
- **Mode A only:** `spec.md` exists. **Mode B:** no `spec.md` — instead the Linear issue body must contain acceptance criteria; if it does not, stop and ask for them to be added in Linear before implementing.
- Cross-module dependencies are IMPLEMENTED (if any)

## Behavior

1. **READ MANDATORY FILES** — If you haven't already, read `sdd/ANTI_PATTERNS.md`, `sdd/QUALITY_GATES.md`, and `AGENTS.md` now. Do NOT skip this.
2. **CREATE / REUSE BRANCH — PRESERVE EXISTING HISTORY** — NEVER work directly on `main`. There are two cases:
   - **Branch does not exist:** create from current trunk:
     ```bash
     git checkout main && git pull origin main   # (or master, whichever this repo uses)
     git checkout -b feature/[ISSUE-ID]
     ```
   - **Branch already exists** (e.g., a prior phase committed `spec.md` / `plan.md` under `docs/issues/[ISSUE-ID]/`, or an A/B-eval setup pre-staged the branch):
     ```bash
     git checkout feature/[ISSUE-ID]            # or whatever the branch is actually named
     git log --oneline                          # confirm prior commits are present
     ```
     **Do NOT `git reset`, `git rebase`, `git commit --amend`, `git rebase -i`, `git push --force` or `git checkout -B` an existing branch.** History is part of the deliverable — losing the spec/plan commit means the merge to main loses those artifacts. Add NEW commits on top; never rewrite.
   All implementation work MUST happen on this branch. Verify branch + log before writing any code (`git branch --show-current && git log --oneline -5`).
3. **UPDATE LINEAR** — Move the issue to **In Progress**. This signals to the team that work is underway. Do NOT skip this step.
   ```bash
   python "$ORCH_HOME/scripts/linear.py" update [ISSUE-ID] --state "In Progress"
   ```
4. **READ** the complete `plan.md` (Mode A: also the complete `spec.md`; Mode B: the Linear issue body's acceptance criteria stand in for the spec)
5. **IMPLEMENT** step by step according to the plan, using **RED/GREEN/VERIFY** for each requirement:

   For each requirement (Mode A: each `RF-XX` in the spec; Mode B: each acceptance criterion in the Linear issue body):
   ```
   a) RED:    Write a test that validates the requirement (it MUST fail — if it passes, the feature already exists)
   b) GREEN:  Implement the minimum code to make the test pass
   c) VERIFY: Run the project's test command (see AGENTS.md). Confirm the new test passes AND no existing tests broke
   d) Only advance to the next RF-XX when VERIFY is OK
   ```

   General rules:
   - Follow the order defined in plan.md
   - If plan.md defines a **dependency graph**, respect parallelization boundaries
   - Respect the project's layer architecture (defined in AGENTS.md)
   - **MODIFY FIRST, CREATE AFTER**: Start with existing files
   - Before creating new file/class, confirm plan.md justifies it
   - Reuse existing packages/modules as planned

   > **Exception**: For pure UI/presentation work (Views, AXAML layouts) where automated testing is impractical, skip RED/GREEN and implement directly. Still run VERIFY (build + existing tests) after each step.

6. **QUALITY GATE** during implementation:
   - Single Responsibility
   - Reuse existing code as planned
   - Tests written alongside code
   - Security verified (apply project's security checklist from AGENTS.md)
   - **LOC limits**: File < 500 (green), 500-600 (yellow: freeze), > 600 (red: split). Method/function < 45 (green), > 55 (split) (AP-05)
   - **Abstractions**: Only with 2+ implementations (AP-01)
   - **Patterns**: Same pattern as rest of project (AP-08)
   - **Dead code**: Zero stubs, placeholders, or empty scaffolds (AP-07)
7. **SELF-CHECK** after each step:
   ```
   [ ] Did I search for existing code before creating? (AP-04)
   [ ] Is complexity proportional to the problem? (AP-02)
   [ ] Is code in the correct layer? (AP-06)
   [ ] Is file within LOC limits? (AP-05)
   [ ] No unnecessary abstractions? (AP-01)
   [ ] Does security have real enforcement? (AP-03)
   ```
8. **INFORM** progress after each step
9. **UPDATE** `sdd/CHANGELOG.md` at the end
10. **COMMIT** — Stage and commit all changes on the feature branch:
   ```bash
   git add -A
   # Unstage local-only pipeline artifacts — they are scratch files the orchestrator reads in
   # place; committing them causes rebase conflicts + dirty-tree blocks + untracked collisions
   # on later phases (check, fix, end-issue, re-merge rounds).
   git reset -q -- .pipeline 'check-report.md' 'smokes*.md' 'end-issue-*.md' 2>/dev/null || true
   git commit -m "[ISSUE-ID]: <short description of what was implemented>"
   ```
   This is MANDATORY. Every implementation session MUST end with a local commit on the feature branch. Do NOT push — that is END-ISSUE. (Add `.pipeline/`, `check-report.md`, `smokes*.md`, `end-issue-*.md` to `.gitignore` if not already excluded.)

## Rules

- **NEVER work on `main`** — all implementation MUST happen on branch `feature/[ISSUE-ID]`
- FOLLOW the approved plan — don't improvise
- DO NOT add unspecified features
- DO NOT refactor code outside scope
- If you find ambiguity: STOP and ask
- If the plan is wrong: STOP and suggest revision
- **ZERO TOLERANCE for build errors, warnings, and test failures**: Run the project's build and test commands (see AGENTS.md). ALL errors, ALL warnings, and ALL failing tests MUST be fixed before considering the task done — even if the failure appears unrelated to your changes or pre-existed. No exceptions.

## Out of Scope — NEVER DO in this phase

- **NEVER** push to remote (that is END-ISSUE step 4)
- **NEVER** create a PR (that is END-ISSUE step 5)
- **NEVER** merge anything (that is END-ISSUE step 6)
- **NEVER** mark the Linear issue as Done (that is END-ISSUE). Moving to In Progress at the START is mandatory (step 3)
- **NEVER** close or comment on GitHub Issues
- **NEVER** run `/sdd-check` or `/sdd-end-issue` — those are separate phases
- **NEVER** delete branches
- **NEVER rewrite git history on an existing branch** — no `git reset` (other than `--mixed` for unstaging your OWN uncommitted changes), no `git rebase`, no `git commit --amend`, no `git checkout -B` of an existing branch, no `git push --force`. If `docs/issues/[ISSUE-ID]/spec.md` or `plan.md` already exists in a prior commit on this branch, that commit MUST survive — add your implementation as new commits on top. (DT-482 incident 2026-05-30: a previous implementation drop spec/plan commit by amending, which would have made the merge to main lose those artifacts.)

**This phase writes code, commits locally, and updates Linear to In Progress.** Branch creation (step 2), Linear update (step 3), and local commit (step 10) are the ONLY git/Linear operations allowed. Do NOT push, create PRs, or mark issues as Done. The workflow continues with CHECK, then END-ISSUE.

## Red Flags — STOP IMMEDIATELY if:

- UI layer with business logic or data access -> Move to service/repository (AP-06)
- Domain/model with infrastructure dependencies -> Remove (AP-06)
- Service with direct data access -> Move to repository (AP-06)
- Code that duplicates existing code -> Reuse (AP-04)
- Abstraction with 1 implementation -> Use direct class/struct (AP-01)
- File > 600 LOC -> Immediate split (AP-05)
- Security for non-sensitive data -> Remove (AP-02)
- Security attribute without enforcement -> Remove or implement (AP-03)
- Empty stub or placeholder -> Implement or don't create (AP-07)
- New pattern for solved problem -> Use existing (AP-08)

## Output

Code implemented according to plan, committed on feature branch, Linear issue In Progress. Ready for CHECK phase.
