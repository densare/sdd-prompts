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

Format: `[ISSUE-ID]` (e.g., DT-48, DS-366 — a Linear issue identifier)

## Linear Access

Issues are tracked in **Linear** (project management tool). To query issues:
- **MCP tools**: If available (e.g., `mcp__linear__get_issue`, `mcp__linear__list_issues`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

## Locate Issue, Spec and Plan

1. **QUERY LINEAR** for `[ISSUE-ID]` to get: title, description, status, labels, and any linked information
2. **FIND** the corresponding `issues.md` in the planning repository (search `projectos/*/requests/em-implementacao/*/issues.md` for the issue ID)
3. From that folder, **READ** `spec.md` and `plan.md`
4. If not found, inform the user

## Pre-conditions

- Linear issue exists and is not Done/Cancelled
- `plan.md` exists and has state PLANNED (approved)
- `spec.md` exists
- Cross-module dependencies are IMPLEMENTED (if any)

## Behavior

1. **READ MANDATORY FILES** — If you haven't already, read `sdd/ANTI_PATTERNS.md`, `sdd/QUALITY_GATES.md`, and `AGENTS.md` now. Do NOT skip this.
2. **CREATE BRANCH** — NEVER work directly on `main`. Create a feature branch:
   ```bash
   git checkout main && git pull origin main
   git checkout -b feature/[ISSUE-ID]
   ```
   All implementation work MUST happen on this branch.
3. **READ** complete spec.md and plan.md
4. **IMPLEMENT** step by step according to the plan, using **RED/GREEN/VERIFY** for each requirement:

   For each RF-XX in the spec:
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

5. **QUALITY GATE** during implementation:
   - Single Responsibility
   - Reuse existing code as planned
   - Tests written alongside code
   - Security verified (apply project's security checklist from AGENTS.md)
   - **LOC limits**: File < 500 (green), 500-600 (yellow: freeze), > 600 (red: split). Method/function < 45 (green), > 55 (split) (AP-05)
   - **Abstractions**: Only with 2+ implementations (AP-01)
   - **Patterns**: Same pattern as rest of project (AP-08)
   - **Dead code**: Zero stubs, placeholders, or empty scaffolds (AP-07)
6. **SELF-CHECK** after each step:
   ```
   [ ] Did I search for existing code before creating? (AP-04)
   [ ] Is complexity proportional to the problem? (AP-02)
   [ ] Is code in the correct layer? (AP-06)
   [ ] Is file within LOC limits? (AP-05)
   [ ] No unnecessary abstractions? (AP-01)
   [ ] Does security have real enforcement? (AP-03)
   ```
7. **INFORM** progress after each step
8. **UPDATE** `sdd/CHANGELOG.md` at the end

## Rules

- **NEVER work on `main`** — all implementation MUST happen on branch `feature/[ISSUE-ID]`
- FOLLOW the approved plan — don't improvise
- DO NOT add unspecified features
- DO NOT refactor code outside scope
- If you find ambiguity: STOP and ask
- If the plan is wrong: STOP and suggest revision
- **ZERO TOLERANCE for build errors, warnings, and test failures**: Run the project's build and test commands (see AGENTS.md). ALL errors, ALL warnings, and ALL failing tests MUST be fixed before considering the task done — even if the failure appears unrelated to your changes or pre-existed. No exceptions.

## Out of Scope — NEVER DO in this phase

- **NEVER** run `git commit` — commits are the responsibility of the human developer, not the AI agent
- **NEVER** run `git add` — staging is also the developer's responsibility
- **NEVER** push to remote (that is END-ISSUE step 4)
- **NEVER** create a PR (that is END-ISSUE step 5)
- **NEVER** merge anything (that is END-ISSUE step 6)
- **NEVER** mark the Linear issue as Done or update its status (that is END-ISSUE step 7)
- **NEVER** close or comment on GitHub Issues
- **NEVER** run `/sdd-check` or `/sdd-end-issue` — those are separate phases
- **NEVER** delete branches

**This phase ONLY writes and modifies files.** No git operations beyond the initial branch creation (step 1). The human developer decides when and what to commit. The workflow continues with CHECK, then END-ISSUE.

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

Code implemented according to plan. CHANGELOG updated. Ready for CHECK phase.
