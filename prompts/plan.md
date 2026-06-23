# SDD Phase: PLAN

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Act as a **Software Architect**. Create an implementation plan from an approved specification.

## Required Context

Read or have access to:
- `sdd/ANTI_PATTERNS.md` — **MANDATORY** anti-patterns to avoid
- `sdd/WORKFLOW.md` — SDD workflow
- `sdd/QUALITY_GATES.md` — mandatory checks (section Gate 2: PLAN)
- `sdd/_templates/plan.md` — plan template
- `AGENTS.md` — project rules, code structure, conventions, layer architecture

## Arguments

Two formats accepted, one per Mode (see §Modes below):

- **Mode A (request planning):** `<project> <request-id>` — e.g. `dentherm DT-G2`. `<request-id>` is the folder name in `projectos/<project>/requests/`.
- **Mode B (issue planning):** `<ISSUE-ID>` — e.g. `DT-589`. Linear issue ID directly.

See AGENTS.md for available projects.

## Modes

`/sdd-plan` runs in one of two **mutually-exclusive modes**. Choose before any audit/spec read.

### Mode A — Request planning (multi-issue, plan lives in planning repo)

- **Trigger:** approved `spec.md` in `planning/aec/projectos/<project>/requests/aprovados/<request-id>/`.
- **Plan path:** `planning/aec/projectos/<project>/requests/em-implementacao/<request-id>/plan.md` (planning repo, **NOT** code repo).
- **Linear:** **creates** N issues + writes `issues.md` with implementation order.
- **Folder move:** `aprovados/` → `em-implementacao/` at the end.
- **Typical scope:** 8–40 pt, multi-component, multiple PRs.
- **Estimate:** estimated **here** per issue created.

### Mode B — Issue planning (single-issue, plan lives in code repo)

- **Trigger:** an existing Linear issue (created via `/sdd-check` deferral, UX review, bug report, follow-up) whose body already contains acceptance criteria.
- **Plan path:** `<code-repo>/docs/issues/<ISSUE-ID>/plan.md` (code repo, near the feature branch).
- **Linear:** **does NOT create** issues — only sets the existing issue's state to `In Progress` and (optionally) attaches the plan link.
- **No `issues.md`, no folder move.**
- **Typical scope:** 1–5 pt, single component, one PR.
- **Estimate:** **do not re-estimate** — the issue already has an estimate set at creation. Plan respects it; if the audit reveals the estimate is wrong, surface that in the audit section but do not silently change the Linear field.
- **Audit Gate 2 inversion:** Linear body did not go through formal `/sdd-specify` approval — audit may legitimately reveal the assumption is wrong/incomplete. When that happens, **document the inversion explicitly** in `plan.md §Audit` + commit message + CHANGELOG (e.g., DT-589 Option A → C4 in `densare/dentherm`).

### Centralised record

This file (`prompts/plan.md` in `densare/sdd-prompts/master`) is the **canonical source** for the mode split. Each consumer repo links to it from `AGENTS.md §SDD Discipline` via the raw URL — propagation is automatic, no per-repo deploy. Repos currently consuming: dentherm, denstudio, densim, denbridge, denretrofit, densolar, denlight.

## Linear Access

Issues are tracked in **Linear**. **Use `scripts/linear.py` for ALL Linear ops** — read, list, create, update, comment, label:

```bash
python "$ORCH_HOME/scripts/linear.py" get DT-NNN
python "$ORCH_HOME/scripts/linear.py" list --team DenTherm --open-only --limit 50 [--json]
python "$ORCH_HOME/scripts/linear.py" create --team DenTherm --title "..." --description "..." --priority 1 --estimate 2 --labels tech-debt
python "$ORCH_HOME/scripts/linear.py" update DT-NNN --state "In Review" --comment "..."
python "$ORCH_HOME/scripts/linear.py" label DT-NNN --add tech-debt
```

`$ORCH_HOME` is exported by the orchestrator; API key auto-discovered from `<sandbox>/.linear.env`.

**DO NOT use Linear MCP** (`mcp__linear__*`). The MCP was detached from sandbox `.mcp.json` on 2026-06-01 — its tool schemas + verbose responses were consuming ~50% of session context. Calling `mcp__linear__*` will fail.

## Deferral Hygiene

If you open a Linear deferral during this phase, the new issue **MUST** carry the `tech-debt` label **plus every label its source issue carries** (a deferral of an `F1` issue is `F1`; an epic child inherits the **epic's** label) — read them with `linear.py get <SOURCE-ISSUE>` → `labels.nodes[].name`. See `SDD_DISCIPLINE.md` §Rule 3 for the full rule and rationale:

**Use the orchestrator helper** `scripts/linear.py` (direct GraphQL, **zero context bloat**) — DO NOT use Linear MCP for batch create/update/label (each MCP call inflates session context):

```bash
python "$ORCH_HOME/scripts/linear.py" create   --team DenTherm --title "<deferral title>"   --description "<why + scope + parent issue link>"   --priority 3 --labels "tech-debt,<each label the source issue/epic carries>"
```

The script auto-creates the `tech-debt` label if missing (color #6e6e6e, description per SDD_DISCIPLINE.md §Rule 3) and applies every `--labels` entry during creation, then prints the new ID.

An unlabelled deferral is a defect — and a deferral that drops its source's scope labels (e.g. `F1`) silently escapes that scope. Carry `tech-debt` **plus** the source's labels so operators' scope/tech-debt filters both stay correct.

## Locate the Spec

**Mode A (request):** Search for `spec.md` in `projectos/<project>/requests/` (in order):
1. `aprovados/<ID>-*/spec.md`
2. `em-analise/<ID>-*/spec.md`
3. `em-implementacao/<ID>-*/spec.md`
4. If not found, inform that spec must be created first

**Mode B (issue):** No separate `spec.md`. The **Linear issue body** plays the role of the spec — fetch it via `python "$ORCH_HOME/scripts/linear.py" get DT-NNN` (§Linear Access). Treat the issue's "Acceptance criteria" / "Fix proposto" sections as the requirements list. If the body lacks acceptance criteria, stop and ask for them to be added in Linear before planning.

## Pre-conditions

- **Mode A:** `spec.md` exists and has state SPECIFIED (approved). If DRAFT, inform that spec needs approval first.
- **Mode B:** Linear issue exists, is in `Backlog` or `Todo` state (not `In Progress`/`Done`), and has acceptance criteria in the body.
- If cross-module dependencies exist: check they are PLANNED or IMPLEMENTED.

## Behavior

0. **AUDIT** (mandatory step before planning):
   - For each point in the request/spec, check if it already exists in code
   - Classify: DONE / PARTIAL / DIVERGENT / NOT STARTED
   - If **DONE**: STOP — feature already exists
   - If **PARTIAL**: Adjust scope to cover ONLY what's missing
   - If **DIVERGENT**: STOP — present divergences to user
   - Record audit result at the top of plan.md

0.5. **TRIAGE `por-triar`** (mandatory, runs on **EVERY** `/sdd-plan` invocation — independent of the request/issue being planned):
   - List the open issues carrying the **`por-triar`** label via `scripts/linear.py` (§Linear Access). These are **ad-hoc** issues created outside the request→spec→plan flow (bugs/findings, legacy-mining, panel results) that still lack an **implementation timing**.
   - For **EACH** such issue, give it a **timing** and then **remove the `por-triar` label**:
     - the **increment `Ix.y`** it belongs to (default — `METODOLOGIA_ARTEFACTOS §3.bis(4)`: an issue that surfaced during an increment belongs to it), **OR** — if it is a defect of the **pre-SDD core** / outside the forward Onda roadmap — the **wave/trigger** that gates it, or the **active effort it blocks** (e.g. add a `blocks` relation to the V&V-parity issue);
     - confirm its anchor exists (request/epic, or `Bug` + `area:*`);
     - then **remove `por-triar`** via `linear.py` (it leaves the queue once it has a "when").
   - Principle: **"zero sem-quando"** alongside **"zero órfãos"** — no issue without a timing. Governance source: `guidelines/METODOLOGIA_ARTEFACTOS.md §3.ter`.

1. **READ** `sdd/ANTI_PATTERNS.md` — keep the anti-patterns in mind
2. **READ** complete spec.md
3. **VALIDATE CLASSIFICATION**: Confirm target repository
4. **INVESTIGATE EXISTING CODE** (Search Before Create):
   - For each requirement, search source code for similar components
   - Decide by order: MODIFY > EXTEND > GENERALIZE > CREATE
   - **RED FLAG**: If plan has more NEW files than MODIFIED files -> STOP and review
5. **CHECK CROSS-MODULE / CROSS-REPO DEPENDENCIES** (see "Multi-Repo Scope" below):
   - If the dependency is satisfiable WITHIN this issue by also changing a sibling repo present in the workspace (e.g. the host/shell repo at `../<repo>/`): do NOT block — add that repo to this plan's scope (see Multi-Repo Scope)
   - For each dependency on a separate, UNMERGED issue, or in a repo NOT present in the workspace (NOT_EXISTS / DRAFT and not doable here): BLOCK
   - For each dependency SPECIFIED (planned / in-flight elsewhere): WARN
6. **ANALYZE** each requirement: map to concrete components, identify risks
7. **QUALITY GATE** — Apply PLAN checklist:
   - Correct layers? (AP-06) — follow project's layer architecture from AGENTS.md
   - Existing code to reuse? (AP-04)
   - Consistent patterns? (AP-08)
   - Interfaces/abstractions with 2+ implementations? (AP-01)
   - Security proportional? (AP-02, AP-03)
   - No planned file > 600 LOC? (AP-05)
   - Zero stubs? (AP-07)
   - Cross-module dependencies satisfied?
   - **Entry-points wired** (AP-09): for EACH entry-point declared in spec, a corresponding file is in "Files to modify" (Navigation.cs, cmd/<svc>/main.go router setup, layout templ, MenuProvider, etc.). If spec declares entry-points but plan does not modify any wiring file → BLOCK.
   - **Storage helpers centralized** (AP-10): for EACH persisted field with 2+ write paths in spec, a centralized helper is planned (or referenced if already exists). The plan names which helper and the conversion it applies.
   - **Last implementation step = wire + navigable smoke**: the implementation order MUST end with a step explicitly named "wire entry-points and run navigable smoke" (or equivalent). This is the seam that catches AP-09 before merge. If the ordered steps end at "create file X" or "implement service Y" without a final wire+smoke step → BLOCK.
8. **ESTIMATE** using Fibonacci scale:
   - 1 = ~30min | 2 = ~1h | 3 = ~2h | 5 = ~4h | 8 = ~8h
   - **Mode A:** estimate each created issue.
   - **Mode B:** **DO NOT re-estimate** — the issue already carries an estimate from creation time. Only surface a mismatch in the audit section (e.g., "audit revealed scope is closer to 2pt than the 1pt on the issue"); do not silently mutate the Linear `estimate` field.
   - >= 13 points: MANDATORY to split (Mode A only — in Mode B, if audit shows the issue is ≥13pt, raise it and stop; do not split unilaterally).
9. **CREATE** `plan.md` with:
   - Target repository
   - Architecture decisions (with justification)
   - Files to create/modify (with layer)
   - Existing packages/modules to reuse
   - Ordered steps (following project's layer order from AGENTS.md)
   - **Dependency graph with parallelization** — for each step, identify what it depends on and what can run in parallel (enables simultaneous work on repos A/B)
   - **Planned tests** — enumerate by category:
     - **Unit / integration tests** (mechanically coverable): list each scenario + assertion. These are run by CI and `/sdd-check` build/test gate.
     - **AUTOMATION-FIRST GATE (mandatory, per smoke):** a human smoke is the most expensive, slowest, regression-prone form of verification — it is the throughput bottleneck and lets silently-broken behavior slip between issues. So for **EVERY** observable behaviour you are about to mark as a manual smoke, you MUST first explicitly ask and answer in the plan: **"Is there an automated test that can verify this instead — to avoid a human smoke?"** Consider, cheapest-first: a **unit test** on the view-model/service; a **headless UI test** (e.g. `Avalonia.Headless` / `[AvaloniaFact]`) that drives the control and asserts the visual tree; **golden-image / snapshot** diffing of a rendered screenshot; **artifact extraction** (PdfPig/pdftotext on a generated PDF, parsing exported XML, diffing a golden file); or a **vision-model-as-judge** over a captured screenshot. For each behaviour write one line: `Automatable? YES → <which automated test + assertion>  |  NO → <why it genuinely needs human eyes>`. **Prefer the automated test and put it under Unit/integration tests.** Reserve a manual smoke ONLY for what genuinely needs human judgement: visual aesthetics/layout, OS-native dialogs (file pickers), real device/hardware, or subjective UX — and even then, name the automated test that partially covers it. (Relates to CLAUDE.md Rule 5.)
     - **Manual smoke tests** (only what survived the automation-first gate above — must enumerate when UI, integration, runtime, or observable behavior genuinely needs human eyes): each smoke is `prerequisite + steps + expected observation + what it validates`. Enumerate **per affected field / per acceptance criterion** — never write generic "edit X → OK → verify" without specifying which fields/scenarios. Scope strictly to THIS issue; do NOT include regression smokes for previously merged issues.
     - **Explicit exclusions** — fields/scenarios deliberately NOT tested in this issue, with justification (e.g., "derived value, no persistence path" / "schema gap deferred to follow-up issue / "covered by separate test fixture X")
     - If acceptance criteria reference observable behavior (e.g., "lista mostra X", "painel abre Y", "valor persiste após reload") and no manual smoke is enumerated, the plan is **incomplete** — `/sdd-check` will reject it.
   - Anti-patterns verified section
   - **Risks and rollback plan** — identify implementation risks with probability/impact/mitigation, and document how to revert if the implementation fails (branch strategy, critical files affected, data reversibility)
   - Cross-module dependencies
   - Total estimate in points
10. **ASK** for confirmation to mark as PLANNED
11. **LINEAR — diverges by mode:**
    - **Mode A:** CREATE LINEAR ISSUE(S). Title, description, estimate, labels. Record issue IDs in plan.md.
    - **Mode B:** issue already exists. Set state → `In Progress`. Do NOT create new issues unless audit splits the scope (rare; ask first).
12. **Mode A only:** `issues.md` is **TEMPORARY STAGING** to draft the issues — NOT a living artifact. Lifecycle: (a) write `issues.md` to compose the issues + order; (b) **SEED them into Linear PROMPTLY** (staging is transient, never a parking lot — an `issues.md` left unseeded makes the work INVISIBLE in Linear); (c) once seeded, **DISCARD the file OR stamp it** at the top: `> **Seeded to Linear: <date>.** ⚠️ Linear is the truth — this file may be stale from this date on. Not a state source.` It MUST NOT carry issue STATE (no "Backlog/Done", no "Linear deferred / local-only" banners) and MUST use REAL Linear IDs (never `DS-CB-01` placeholders). The `ID→folder` anchor moves to a Linear **label `request:<id>`**; the implementation order lives in **`plan.md` (§Order)** — neither needs a parallel file that rots.
13. **Mode A only:** MOVE request folder to `em-implementacao/`.
14. **Mode A only:** UPDATE `projectos/<project>/requests/README.md`.

## Planning vs Implementation Separation

The split depends on mode:

- **Mode A** lives entirely in the **planning repo** until handoff:
  ```
  AUDIT -> SPECIFY -> PLAN (in planning repo) -> Linear issues -> END (handoff to code repo via /sdd-implement)
  ```
- **Mode B** lives in the **code repo** from the start (the issue itself is the spec; the plan is a development artifact near the branch):
  ```
  Linear issue (pre-existing) -> PLAN (in code repo docs/issues/<ID>/) -> /sdd-implement -> /sdd-check -> /sdd-end-issue
  ```

In both modes, this prompt **does not write production code** — only plan and document.

## Multi-Repo Scope

An issue MAY legitimately require changes in **more than one repo** — e.g. a module repo AND the host/shell repo it plugs into (a sibling checkout at `../<repo>/`), or vice-versa. This is **allowed**: one issue may change multiple repos.

- A cross-repo need is **NOT** an automatic BLOCK and **NOT** a reason to split the work into a second issue. If doing the issue correctly requires the host to expose, publish, or refresh something the module consumes (or vice-versa), **include that change in THIS plan** and scope **both** repos.
- Only **BLOCK** when the dependency is genuinely unavailable — it lives in a repo **not present in the workspace**, or it depends on a **separate, unmerged issue** — never merely because the change is "in the other repo" (e.g. "that package is consumed as a built artifact" is not a blocker when its source repo is checked out beside this one and can be rebuilt/republished).
- In `plan.md`, the **Target repository** and **Files to create/modify** sections list **every** repo touched (group files by repo, with the sibling path). The ordered steps sequence the repos correctly — e.g. a host change + (re)publish of any consumed package BEFORE the module change that depends on it — and the final navigable smoke exercises the end-to-end result across repos.

## Rules

- DO NOT write code — only plan and document
- **SEARCH BEFORE CREATE** (AP-04)
- **BLOCK** duplicating existing code (AP-04)
- **BLOCK** abstraction with 1 implementation without justification (AP-01)
- **BLOCK** planned file > 600 LOC (AP-05)
- **BLOCK** spec declares entry-points but plan does not wire them (AP-09)
- **BLOCK** spec declares persisted field with 2+ write paths but plan has no centralized helper (AP-10)
- **BLOCK** implementation order that does not end with "wire entry-points + navigable smoke"
- **DO NOT BLOCK** on a cross-repo dependency that is doable within this issue (the other repo is checked out beside this one) — scope **both** repos instead (see Multi-Repo Scope). BLOCK only when it needs a separate, unmerged issue or a repo absent from the workspace.
- >= 13 points: MANDATORY to split

## issues.md Template (TEMPORARY staging — discard or stamp after seeding)

`issues.md` is **temporary staging** to draft the issues before they go to Linear. After seeding, **discard it OR stamp it stale** (see step 12). It is NOT a manifest, NOT a tracker, carries **NO state**, uses **real** Linear IDs. The `ID→folder` anchor lives in a Linear label `request:<id>`; the implementation order lives in `plan.md`.

```markdown
# Issues (staging): <ID> - <Title>

> **Seeded to Linear: <date>.** ⚠️ Linear is the truth — may be stale from this date. Not a state source. (Omit this line only while still un-seeded; delete the file once seeded if you prefer.)

## Linear Issues

| # | Issue | Title | Repository |
|---|-------|-------|------------|
| 1 | [XX-NNN](linear-url) | <title> | <repo> |
```

Do **not** add a status column or any "Linear deferred / local-only / IDs = —" banner — those rot and mislead (a stale "Linear deferred" banner once made an audit conclude issues were local-only when they were already in Linear). Do **not** leave an un-seeded `issues.md` lingering — seed promptly or the work is invisible in Linear.

## Output

- **Mode A:** folder in `planning/aec/projectos/<project>/requests/em-implementacao/<request-id>/` with `request.md` + `spec.md` + `plan.md` + `issues.md`. Linear issues **created**.
- **Mode B:** single file `<code-repo>/docs/issues/<ISSUE-ID>/plan.md`. Linear issue **state → In Progress** (existing issue, no new ones created).
