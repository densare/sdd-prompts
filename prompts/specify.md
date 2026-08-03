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
   - Acceptance criteria (Given/When/Then)
   - Unconsidered edge cases
   - Technical dependencies
   - Cross-module dependencies
   - The explicit risk tier and its provenance (see **Risk Classification** below)
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
   - Risk is explicitly declared, justified, and adjusted for irreversibility?
11. **CLARIFY** with user if necessary
12. **ASK** for confirmation to mark as SPECIFIED

## Risk Classification (RT-01 — Mandatory)

`/sdd-specify` declares the risk of the unit being specified. If one spec covers several components, use
the highest tier among the components in scope. Absence is not low risk.

| Tier | Use when the work touches |
|------|---------------------------|
| `t1` | Cryptography; regulated calculation engines; schema migrations; fail-closed paths; multi-tenancy; billing; authentication/authorization; or content moderation. |
| `t2` | New business logic or contracts between repositories/modules. This is the conservative default when uncertain; a `t2` chosen under uncertainty is `declared`, never `fallback`. |
| `t3` | UI-only behavior, documentation, mechanical refactors, or tests, provided no `t1`/`t2` mechanism is touched. |

Apply all of these rules:

1. **Declare, never infer `t3`.** If classification cannot be established from evidence, use `t2`. Record the classification as declared work; a downstream default/fallback is absence of classification and MUST NOT reduce scrutiny.
2. **Apply the irreversibility modifier.** Ask: _if a defect is found after this artefact is emitted, do all existing copies disappear or get replaced by a redeploy?_ If **no**, raise the tier by one step (`t3→t2`, `t2→t1`; `t1` stays `t1`). Raise the component that **emits** the durable artefact — and also the **last non-trivial transformer** before it, when that transformer can alter, omit, truncate, paginate or mis-encode what was already approved. Do NOT raise consumers, adapters or intermediate contracts that merely carry the intent. The cut is on the **business** chain, not the **technical** one: a tier governs the scrutiny of the **work that lands on that component**, not a test plan — a change to a shared renderer is born as an issue of the shell and never touches an issue of the module that delivers, so the module's `t1` does not protect it. Whoever pays the scrutiny must be the component where the work lands. This does **not** apply to private re-emittable exports, previews, drafts, or files replaceable in the customer's own account. If uncertain whether the artefact is irreversible, do **not** apply this modifier — the base classification already fails upward, and making both axes fail upward recreates tier inflation.
3. **Persist the source.** The spec header MUST carry `Risco: t1|t2|t3` and `Proveniencia: declared|inherited`. In cap-first projects, also write `risk: t1|t2|t3` in the capability metadata and preserve provenance for the carve (`declared` when confirmed for the concrete issue, `inherited` when only inherited from the capability). `fallback` means nothing was declared and the engine defaulted to `t3`; it is absence of classification and never qualifies for reduced scrutiny.
4. **Runtime can only raise risk when the mechanism exists.** The Forge diff reclassifier runs post-commit and pre-gate: files outside the manifest, new schema/persistent formats, auth/authz/crypto/PII, regulated paths, cross-repo changes, removed, ignored or weakened tests, new public interfaces, new dependencies, or an oversized diff raise `effective_risk`. It MUST never lower the declared tier. If a project's runtime does not implement this reclassifier, the declared tier is the only control — classify accordingly rather than assuming a later safety net.

For a capability whose tier is higher than a concrete issue's apparent work, do not silently downgrade it. A later carve may assign a lower issue tier only with a reason declared in the increment contract, and never below `t2` when the issue touches the mechanism that made the capability `t1`. Without that reason the carve is invalid; absence is never read as `t3`.

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
- If the spec's risk tier or provenance is absent: BLOCK — do NOT mark SPECIFIED. Never treat missing risk as `t3`; use a declared `t2` while uncertain.
- Risk declared during specification is the **default** for issues carved from it, not a floor. A carve may lower an issue's tier only with a reason declared in the increment contract, and never below `t2` when the issue touches the mechanism that made the capability `t1`. The post-diff runtime reclassifier may only maintain or raise.

## Output

### Single Repository Task
Folder `<ID>-<name>/` created with `request.md` + `spec.md` (state DRAFT or SPECIFIED).

### Cross-Module Task (2+ Repositories)
Files created inside the folder:
- `spec.md` — index spec that lists sub-tasks and dependencies
- One spec per module (e.g., `spec-core.md`, `spec-app.md`)

Inform implementation order: shared/core modules first, then dependent modules.
