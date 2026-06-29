# SDD Phase: SPECIFY

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Act as a **Requirements Analyst**. Translate user request into a technical specification.

## Required Context

Read or have access to:
- `sdd/ANTI_PATTERNS.md` — **MANDATORY** anti-patterns to avoid
- `sdd/WORKFLOW.md` — SDD workflow
- `sdd/QUALITY_GATES.md` — mandatory checks (section Gate 1: SPECIFY)
- `sdd/_templates/spec.md` — spec template
- `AGENTS.md` — project rules, module classification, conventions

## Arguments

Format: `<project> <ID>` — see AGENTS.md for available projects and ID format.

## Deferral Hygiene

If you open a Linear deferral during this phase (specify often surfaces out-of-scope work that becomes a new issue), the new issue **MUST** carry the `tech-debt` label **plus every label its source issue carries** (a deferral of an `F1` issue is `F1`; an epic child inherits the **epic's** label) — read them with `linear.py get <SOURCE-ISSUE>` → `labels.nodes[].name`. See `SDD_DISCIPLINE.md` §Rule 3 for the full rule and rationale:

**Use the orchestrator helper** `scripts/linear.py` (direct GraphQL, **zero context bloat**) — DO NOT use Linear MCP for batch create/update/label (each MCP call inflates session context):

```bash
python "$ORCH_HOME/scripts/linear.py" create   --team DenTherm --title "<deferral title>"   --description "<why + scope + parent issue link>"   --priority 3 --labels "tech-debt,<each label the source issue/epic carries>"
```

The script auto-creates the `tech-debt` label if missing (color #6e6e6e, description per SDD_DISCIPLINE.md §Rule 3) and applies every `--labels` entry during creation, then prints the new ID.

An unlabelled deferral is a defect — and a deferral that drops its source's scope labels (e.g. `F1`) silently escapes that scope. Carry `tech-debt` **plus** the source's labels so operators' scope/tech-debt filters both stay correct.

## Locate the Request

Search for the request file in `projectos/<project>/requests/` (in order):
1. `aprovados/<ID>-*.md` (simple file) — preferred
2. `em-analise/<ID>-*.md` (simple file)
3. `em-implementacao/<ID>-*/request.md` (already converted to folder)
4. If not found, inform the user they should create the request first

## Behavior

1. **READ** `sdd/ANTI_PATTERNS.md` — keep the anti-patterns in mind
2. **READ** the complete `request.md` — it's the user's voice, do not alter
3. **SEARCH BEFORE CREATE**: Look for existing code that does something similar (AP-04). Explicitly list what you found.
4. **CLASSIFY** the functionality using the project's module classification (defined in AGENTS.md):
   - Key question: "Who needs this functionality?" — determines which module/repository
   - If classification implies a different repository than expected: ALERT
5. **CHECK CROSS-MODULE DEPENDENCIES**:
   - Does this request belong to the module where it's stored, or should it be in another?
   - Does this request need functionality that exists in another module?
   - Does this request touch 2+ repositories?
     - If yes -> SPLIT into sub-tasks per repository
     - Create separate spec for each sub-task
     - Sequence: shared/core modules first, then dependent modules
6. **CREATE** spec.md:
   - If the request is a simple file: convert to **folder**, move file to `<ID>-<name>/request.md`, create `spec.md` inside
   - If already a folder: create `spec.md` inside
   - Use the template `sdd/_templates/spec.md`
7. **ANALYZE** and identify:
   - Implicit requirements in the request
   - Ambiguities to clarify with the user
   - **Map RF ↔ CA bidirectionally**: every RF references the request CA it satisfies (`Satisfaz: CA-NN`); every request CA maps to an RF. **No RF without a CA** (untestable requirement) and **no orphan CA**. Carry each CA's **Prova** line (test / regression-oracle / benchmark / smoke + fixture) into the spec — it is what `/sdd-check` will execute. **1 CA = 1 obligation** (split composite CA).
   - **Regulated calculation** (legally-valued result): require a non-regression CA (full oracle suite within approved tolerance; oracles by stable ID; zero tolerance on a verdict crossing a legal limit; compare intermediate parts, not only the total). This maps to the `validate-against-legacy` path in `/sdd-check`.
   - Unconsidered edge cases
   - Technical dependencies
   - Cross-module dependencies
8. **DECLARE ENTRY-POINTS** (AP-09):
   - Is this feature observable by the user / external client (UI, HTTP route, message subscriber)?
   - If yes: enumerate ALL paths the user/client uses to reach the feature in the spec's "Entry-Points" section (menu item, nav node, keybinding, HTTP route, hx- attribute target, event subscriber). For each entry-point: type, location (file:line if known), and whether it already exists or must be created/modified.
   - Write the **1-line navigable smoke**: the path from fresh app/deploy state to the feature. This becomes a mandatory smoke at `/sdd-check`.
   - If feature is purely internal (helper, refactor, shared package) with no external observability: mark "Entry-Points: N/A" and justify in 1 line.
   - If spec adds RFs observable by user but `Entry-Points` table is empty → spec is incomplete; do NOT mark SPECIFIED.
9. **DECLARE STORAGE SEMANTICS** (AP-10):
   - Does the feature persist state (DB, file, settings, in-memory survives reload)?
   - If yes: for EACH persisted field touched, fill the spec's "Storage Semantics" table with `field | canonical stored form | conversion in each write path | conversion in read path`.
   - If 2+ write paths exist for the same field: a single centralized helper must be planned (name it explicitly).
   - If the feature CHANGES the canonical form of an existing field: ALERT, require migration plan.
   - Add the 4 mandatory round-trip edge cases (EC-RT1..RT4: no-op edit, save→reopen→save, import-vs-new, mass-update preserves untouched fields) unless any is justifiably N/A.
   - If feature persists state but `Storage Semantics` table is empty → spec is incomplete; do NOT mark SPECIFIED.
10. **QUALITY GATE** — Apply SPECIFY checklist:
   - Does task do ONE thing?
   - Does similar code exist? (AP-04)
   - Is there premature generalization? (AP-01)
   - Doesn't mix multiple responsibilities?
   - Is security proportional to data type? (AP-02)
   - Cross-module dependencies identified and documented?
   - Entry-points enumerated for every user-observable RF? (AP-09)
   - Storage semantics declared for every persisted field? (AP-10)
   - Round-trip edge cases (EC-RT1..RT4) present or justifiably absent? (AP-10)
   - **Every RF maps to ≥1 CA and every request CA maps to an RF (bidirectional)? Every CA has a Prova line? (no Prova = vague CA, BLOCK)**
   - **Blocking non-functional needs (performance/security/local-format) expressed as CA, not buried in prose?**
11. **CLARIFY** with user if necessary
12. **ASK** for confirmation to mark as SPECIFIED

## Rules

- DO NOT discuss implementation (that's for PLAN phase)
- Focus ONLY on expected behavior
- ALERT if spec mixes multiple responsibilities
- Keep `request.md` original UNCHANGED
- If task touches 2+ repositories: MANDATORY to split into sub-tasks
- If task depends on non-existent functionality in another module: ALERT
- If feature is user-observable but `Entry-Points` section is empty: BLOCK (AP-09)
- If feature persists state but `Storage Semantics` section is empty: BLOCK (AP-10)
- The navigable smoke (1 line under Entry-Points) IS the seed of the manual smoke that `/sdd-check` will demand. Write it as if the reviewer will execute it without reading code.

## Output

### Single Repository Task
Folder `<ID>-<name>/` created with `request.md` + `spec.md` (state DRAFT or SPECIFIED).

### Cross-Module Task (2+ Repositories)
Files created inside the folder:
- `spec.md` — index spec that lists sub-tasks and dependencies
- One spec per module (e.g., `spec-core.md`, `spec-app.md`)

Inform implementation order: shared/core modules first, then dependent modules.
