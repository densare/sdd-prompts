# SDD Phase: CHECK

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Act as a **Code Reviewer**. Verify if the implementation meets the technical specification.

## Required Context

Read or have access to:
- `sdd/ANTI_PATTERNS.md` — **MANDATORY** anti-patterns to check
- `sdd/QUALITY_GATES.md` — mandatory checks (section Gate 4: CHECK and Gate 5: TESTS)
- `AGENTS.md` — project rules, conventions, layer architecture, security checklist

## Arguments

Format: `[ISSUE-ID]` (e.g., DT-48, DS-366 — a Linear issue identifier)

## Linear Access

Issues are tracked in **Linear**. **Use `scripts/linear.py` for ALL Linear ops** — read, list, create, update, comment, label:

```bash
python "$ORCH_HOME/scripts/linear.py" get DT-NNN                                          # full issue (description, labels, parent/children, relations)
python "$ORCH_HOME/scripts/linear.py" list --team DenTherm --open-only --labels F1 --limit 50 [--json]
python "$ORCH_HOME/scripts/linear.py" create --team DenTherm --title "..." --description "..." --priority 1 --estimate 2 --labels tech-debt
python "$ORCH_HOME/scripts/linear.py" update DT-NNN --state "In Review" --comment "..."
python "$ORCH_HOME/scripts/linear.py" label DT-NNN --add tech-debt
```

`$ORCH_HOME` is exported by the orchestrator; API key auto-discovered from `<sandbox>/.linear.env`.

**DO NOT use Linear MCP** (`mcp__linear__*`). The MCP was detached from sandbox `.mcp.json` on 2026-06-01 — its tool schemas + verbose responses were consuming ~50% of session context. Calling `mcp__linear__*` will fail. If linear.py is missing an op, extend it — don't fall back to MCP or curl.

## Locate Issue and Spec

1. **QUERY LINEAR** for `[ISSUE-ID]` to get: title, description, status
2. **FIND** the corresponding `issues.md` in the planning repository (search `projectos/*/requests/em-implementacao/*/issues.md` for the issue ID)
3. From that folder, **READ** `spec.md` and `plan.md`
4. If not found, inform the user

## Independent Review Rule

> **/sdd-check MUST run in an independent context** — never in the same session that ran /sdd-implement.
> This eliminates the implementer's bias and provides a genuine "second pair of eyes".

How to ensure independence:
- **Preferred**: Run /sdd-check in a **new Claude session** (fresh context, no memory of implementation decisions)
- **Alternative**: Run on the **alternate repo copy** (e.g., implement on repo A, check on repo B after git pull)
- **Minimum**: If same session is unavoidable, explicitly state this in the report header as `⚠️ Same-session review`

## Behavior

1. **READ** complete spec.md
2. **ANALYZE** implemented code (approach it as if seeing it for the first time)
3. **VERIFY** each functional requirement:
   - Was it implemented? (concrete evidence)
   - Acceptance criteria met?

4. **QUALITY GATE** — Apply CHECK checklist:

### Architecture
- [ ] Code is in the correct layer? (follow project's layer architecture from AGENTS.md)
- [ ] Dependencies respect direction? (layers only depend downward)
- [ ] Shared code is in the shared module?
- [ ] UI layer only does: binding, commands, call services?
- [ ] Domain/model doesn't depend on infrastructure?

### Over-engineering (AP-01, AP-02)
- [ ] COUNT: How many abstractions? How many with only 1 implementation?
- [ ] Each 1-implementation abstraction has documented justification?
- [ ] No abstractions "for the future"?
- [ ] Complexity proportional to the problem?
- [ ] Security proportional to data type?

### Duplication (AP-04)
- [ ] Search for duplicated code between modules/packages
- [ ] Shared code is in the shared module?
- [ ] No copy-paste between modules?

### Real Security (AP-03)
- [ ] Apply the project's security checklist (from AGENTS.md)
- [ ] EACH security measure has verifiable enforcement?
- [ ] No redundant security layers?
- [ ] No decorative attribute/annotation without handler?

### Patterns and Consistency (AP-08)
- [ ] Single pattern per problem?
- [ ] Consistent with the rest of the project?
- [ ] Follows project conventions (from AGENTS.md)?

### Dead Code (AP-07)
- [ ] Zero empty stubs or placeholders?
- [ ] Everything created is actually used?

### Complexity Limits (AP-05)
- [ ] No file > 600 LOC (red zone)?
- [ ] No method/function > 55 LOC (red zone)?
- [ ] No class/module > 15 public methods?

### Build and Tests — ZERO TOLERANCE
- [ ] Build produces ZERO errors and ZERO warnings? (run project's build command from AGENTS.md)
- [ ] Tests produce ZERO failures? (run project's test command from AGENTS.md)
- [ ] If any error, warning, or test failure exists — even if apparently unrelated or pre-existing — it MUST be fixed before proceeding. No exceptions.
- [ ] Tests cover spec requirements?
- [ ] Tests verify OUTPUT, not internal state?
- [ ] Test scenarios are realistic?

### Manual Smoke Tests — operator-driven verification (MANDATORY GATE)

> **Smoke is part of `/sdd-check`, NOT `/sdd-end-issue`.** Acceptance must be confirmed here before APPROVED verdict. `/sdd-end-issue` is purely close-out machinery (push/PR/merge/Linear) and assumes acceptance was already validated.

#### Pre-step — Finalize mode (read operator's existing smoke responses)

If `smokes.<ISSUE>.md` already exists in the sandbox root AND at least one `**Resposta:**` line is **not** a placeholder (placeholder = wrapped in `_(...)` underscores), the operator has already responded to a prior smoke prompt. In that case:

- **READ** `smokes.<ISSUE>.md` and parse per-smoke responses (PASS / FAIL / DEFERRAL).
- **DO NOT regenerate** the smoke list/file. Do not write `## SMOKE TEST REQUIRED` again. Do not rewrite per-smoke blocks. Operator's authored content is the source of truth.
- **AGGREGATE** the verdict:
  - All PASS → final verdict = **APPROVED** (or APPROVED_WITH_DEFERRALS if other gates flagged non-blocking items).
  - Any DEFERRAL + remaining PASS → final verdict = **APPROVED_WITH_DEFERRALS** (operator accepted deferred items as separate Linear issues — file them now per Deferral Hygiene if any are new).
  - Any FAIL → final verdict = **REJECTED** with the FAIL reason copied into the report's corrective-actions section.
  - Any PENDING (`**Resposta:** _(... underscores ...)_` placeholder still present in some smoke) → operator hasn't finished; emit verdict **SMOKE_REQUIRED** but keep the existing file unchanged.
- Update `check-report.md` with the §Smoke Aggregate section (smoke counts + each smoke's verdict + observations). Do not overwrite check-report.md if it was already approved/rejected — append a "re-finalize" timestamped section.
- Emit the final verdict to `result.json` so the watcher advances normally (`next_phase_for(check, APPROVED) → end-issue`; `next_phase_for(check, REJECTED) → fix`).
- **Skip Step 0 onwards** (no new smoke list, no Legacy Validation re-run unless explicitly asked).

Why this exists: previously check would always regenerate `smokes.<ISSUE>.md` on re-spawn, overwriting the operator's `**Resposta:** PASS` edits and looping the issue back to SMOKE_REQUIRED forever (DT-690 incident 2026-06-02). Operator's signal `"resposta dada do XXX"` triggers a check re-run — the re-run must HONOR existing responses, not erase them.
>
> **Anti-pattern observed in practice:** deferring smoke as "deferral #1 → part of end-issue checklist". Result: smoke fails during end-issue → STOP/REJECT/restart cycle (e.g., DT-576, DT-577). Correct workflow: smoke during check, end-issue mechanical.

#### Step 0 — Choose validation path (Legacy diff vs Human smoke)

Before running the smoke list, query the Linear issue's labels (`python "$ORCH_HOME/scripts/linear.py" get <ISSUE-ID>` → `labels.nodes[].name`). Pick the path based on the label set:

- **Label `validate-against-legacy` present** → take the **Legacy Validation Path** (below). This replaces human smoke entirely. Use for issues where correctness is "produces identical output to a legacy reference implementation" — typically XML/JSON exporters, mappers, parsers, calculators where the legacy code is the oracle. Operator decision 2026-06-02: human eyes are wrong for these; a diff against legacy is.
- **Label `validate-against-legacy` ABSENT** → take the Human Smoke Path (Step 1 onwards).

##### Legacy Validation Path

Goal: prove the new implementation produces output equivalent to the legacy implementation on the same input(s).

1. **Identify the legacy reference** — from the Linear issue body, `plan.md`, or `AGENTS.md`. Typical locations: `D:/dev/eac/dentherm_legacy/` (Java legacy). The issue body should name the specific legacy class/method (e.g. `XmlExporterRecs.java:1430-1525`); if it does not, **STOP** and ask the operator before guessing.
2. **Pick a representative input** — a known-good project file (e.g. `validation/cases/process1/process1.dtz`, `bicas.dtz`, `susana.dtz`) OR a fixture the spec/plan names. If the issue is generic (no input specified), use **all three references** (process1 + bicas + susana) for coverage.
3. **Produce two outputs**, on the same input:
   - **NEW output**: invoke the new code path (export, mapper, calculator — whatever the issue introduces). Capture to a file in `.pipeline/legacy-diff.<ISSUE>/new.<ext>`.
   - **LEGACY output**: invoke the legacy code path on the same input. Capture to `.pipeline/legacy-diff.<ISSUE>/legacy.<ext>`. If the legacy path is JVM-based, invoke via `java -cp ...` from the legacy repo; if CLI, use the CLI; if it requires a database or service, document the limitation and ask operator for the pre-generated legacy output.
4. **Diff** the two outputs:
   - For XML: normalise namespaces + whitespace + attribute order before diff. Use a structural XML diff (e.g. `xmldiff`, `diff -u` on canonicalised XML, or a project helper if one exists). Report differences as `<XPath>: expected=<legacy> got=<new>`.
   - For JSON: canonicalise key order, then `diff` on sorted JSON.
   - For numeric calculation outputs: per-field comparison with explicit tolerance (typical 0.01 absolute or relative — match what unit tests use); flag any field exceeding tolerance with `field, expected=<v>, got=<v>, delta=<d>`.
   - For free-form text: line-by-line diff acceptable but flag every difference.
5. **Verdict** is mechanical:
   - **Empty diff (zero differences)** → record `**Legacy diff:** PASS (N fields/lines compared, zero divergence)` in the check report under a new heading `## LEGACY VALIDATION`. The check report's final verdict is APPROVED (or APPROVED_WITH_DEFERRALS if other gates flagged non-blocking items).
   - **Non-empty diff** → record the full diff verbatim under `## LEGACY VALIDATION` with `**Legacy diff:** FAIL`. Final verdict is REJECTED. The fix phase will read the diff and address each divergence.
   - **Cannot run legacy** (env missing, JVM unavailable, legacy code path broken) → record `**Legacy diff:** BLOCKED — <reason>` and verdict REJECTED with a note that operator must either fix the legacy invocation or convert this issue to human-smoke (remove `validate-against-legacy` label, add canonical smokes to plan).
6. **Do NOT request human smoke** when this path is taken. The diff IS the smoke. The orchestrator advances on the report's final verdict; no `## SMOKE TEST REQUIRED` section is emitted.
7. **Persist artifacts** — keep `.pipeline/legacy-diff.<ISSUE>/{new,legacy,diff}.<ext>` on the branch so the fix phase (and future audits) can inspect what was compared. Add them to git (small text files; XML/JSON are usually < 1 MB).

Continue to "Anti-patterns" gate after recording the LEGACY VALIDATION result. Skip Step 1 onwards.

#### Step 1 — Locate the smoke list

- [ ] Read `plan.md` §"Planned tests" → "Manual smoke tests" / "Smoke manual" list.
- [ ] If the list is **empty** AND the issue body has observable acceptance criteria (e.g., "lista mostra X", "painel abre Y", "edit field → value persists"):
  - This is a **plan deficiency**. Flag in `Spec Deviations`.
  - **STOP** and derive smokes from the acceptance criteria yourself, then proceed to Step 2 (do NOT skip smoke gate just because plan is incomplete).
- [ ] If both empty (no smokes planned AND no observable acceptance criteria — e.g., pure refactor with unit-test coverage): document as "Smoke: N/A (no observable behavior)" in the report and skip to anti-patterns.

#### Step 2 — Present the smoke list to the operator as numbered step-by-step recipes (one per smoke)

- [ ] **STOP all further verification work.** Before producing the check report, ask the operator to run the smokes. Present each smoke as a **separate block with a bold heading and a vertical numbered list** (one step per line). Do NOT use a markdown table with line-break trickery (`<br>` inside cells does not render line breaks in most terminal markdown renderers — steps collapse into a paragraph).

##### Format template (MANDATORY — exact structure, no variants)

> The orchestrator parses smoke sections automatically (`scripts/read_smokes.py` + `surface_smoke.py`).
> Any deviation from the structure below silently breaks parsing and forces a manual canonical
> rewrite by the operator before the smoke can be processed. Recurring incidents 2026-05-30
> (DT-660, DT-661, DT-482 ag3) all traced to this. The format below is **canonical** — emit it
> verbatim, do not paraphrase headings, do not wrap in code blocks, do not use flat numbered
> lists across all scenarios.

```markdown
## SMOKE TEST REQUIRED

<one short paragraph: why human eyes are required (UI/visual/runtime) and what data setup applies>

---

**Smoke 1 — <short scenario title>**
**Runner:** HUMAN

1. <setup action, e.g. "Importar `process1.dtz`">
2. <navigate, e.g. "Abrir painel Envolvente">
3. <interact, e.g. "Duplo-clicar uma parede">
4. <change, e.g. "Tab **Geometria** → dropdown Orientation → escolher **Norte**">
5. <commit, e.g. "Clicar **OK**">
6. **Deves ver** <observable outcome, e.g. "`Az: Norte` na linha da parede">

**Resposta:** _(operator writes PASS | FAIL | DEFERRAL — see response rules below)_

---

**Smoke 2 — <next scenario title>**
**Runner:** AGENT

1. <setup, e.g. "Run `go run . extract` in `scripts/migrate-dentherm`">
2. <act, e.g. "Open `canonical/license_items.json`">
3. **Expect** <machine-checkable outcome, e.g. "row with payment has `payment_id`; offer row omits it">

**Resposta:** _(operator writes PASS | FAIL | DEFERRAL)_

---
```

##### Authoring rules (strict)

- **Section header is literally `## SMOKE TEST REQUIRED`** — no numbering prefix (`## 6. SMOKE TEST...` breaks `surface_smoke.py` extraction), no translation, no variants like "## Manual Smokes".
- **Each smoke is its own block** with bold heading `**Smoke N — title**` (em-dash or hyphen, both accepted). Do NOT collapse all scenarios into a single code block or a single flat numbered list across the section — those formats are unparseable.
- **`**Runner:**` line immediately after each `**Smoke N — title**` heading**, on its own line, value `AGENT` or `HUMAN`. This tells the orchestrator who executes the smoke:
  - **`HUMAN`** — requires human eyes/hands: GUI interaction, visual layout/rendering, "looks right" judgement, desktop-window (Avalonia) flows, anything where the acceptance is *what the operator sees on screen*. **This is the default** — if a smoke is even partly visual, or you are unsure, mark it `HUMAN`.
  - **`AGENT`** — fully machine-checkable with no visual judgement: CLI commands and their exit status / stdout, file contents (JSON/CSV) inspected programmatically, SQL queries and row counts, HTTP requests and response codes/bodies, log assertions. A headless agent can run these and decide PASS/FAIL deterministically.
  - **Rule of thumb:** if the expected observation can be expressed as "this command/query returns this value" → `AGENT`. If it is "the operator sees X on the screen / the layout looks Y" → `HUMAN`. Mixed flows split into separate smokes by runner where possible; if inseparable, mark `HUMAN`.
  - **CRITICAL — "needs infrastructure" is NOT "needs a human".** A smoke that requires a database, a running service, seed data, a JWT, or `docker compose up` is still `AGENT` if its verdict is machine-checkable. The headless runner can stand infra up (compose, migrations, seeds) and, if it genuinely cannot, reports `BLOCKED-INFRA` — it does not need a person. Classify by *how the result is judged* (a query/count/log → AGENT; a visual/UX judgement → HUMAN), NOT by *what the test depends on*. Example: "run `extract`, open `canonical/licenses.json`, expect every row has `legacy_product_id`" is `AGENT` even though it needs the legacy MySQL DB. Marking such smokes `HUMAN` wrongly parks the issue waiting for an operator who has nothing visual to look at.
  - **Backward compatibility:** a smoke with no `**Runner:**` line is treated as `HUMAN` (the historical default). Existing reports without the tag keep working unchanged.
- **Per-smoke numbered list** (`1.`, `2.`, ...): one step per line. Renders correctly in terminals and browsers without `<br>` hacks.
- The expected observation is **the last numbered step**, prefixed with `**Deves ver**` (or `→ expected:` in English projects).
- After the steps, a literal `**Resposta:**` line with an italic placeholder — the operator fills this with `PASS`, `FAIL — <reason>`, or `DEFERRAL — <Linear-ID> <reason>`. The placeholder is what the parser uses to detect "answer slot present but pending".
- `---` horizontal rule between smokes for visual separation (also a robust block delimiter for the parser).
- Steps must be **concrete**: name the file, tab, control, value. Avoid generic phrasings ("edit the field") — say "Tab Geometria → dropdown Orientation → escolher Norte".
- Each smoke MUST be **per-field or per-flow**, not generic. Generic smokes ("edit element + OK") can mask field-specific gaps. If the change touches 4 fields, present 4 smokes.
- Include the test data context in step 1 of the first smoke (e.g., "Importar `process1.dtz`" or "Abrir projecto com PEN + ENU"). Subsequent smokes can assume the same project is open.

##### Anti-patterns that BREAK the parser (do not emit)

These shapes have all been observed and required manual rewrites — do not use them:

1. ❌ **Single fenced code block wrapping all smokes** (` ```\nSmoke 1\n...\nSmoke 5\n``` `): parser can't see headings. Seen in DT-660.
2. ❌ **Flat numbered list across the whole section** (`1. <scenario 1 step 1>` ... `15. <scenario 5 step 3>`) without `**Smoke N — title**` bold dividers. Parser detects zero smokes. Seen in DT-661.
3. ❌ **Section header with numbering prefix** (`## 6. SMOKE TEST REQUIRED`): `surface_smoke.py`'s regex tolerates this *now* but it makes downstream parsers brittle. Use plain `## SMOKE TEST REQUIRED`. Seen in DT-482 ag3.
4. ❌ **Smoke heading as H3** instead of bold (`### Smoke 1`): tolerated, but bold is the canonical form — pick one and stick to the template above.
5. ❌ **Missing `**Resposta:**` slot per smoke**: operator can still respond, but the parser can't tell "pending" from "PASS" reliably. Always include the slot.

##### Why no table, no code block

Markdown tables collapse multi-line cells in most renderers (terminal CLIs, GitHub markdown viewer with default CSS, plain-text logs). Even when `<br>` works in HTML output, it doesn't in plain markdown rendering. Numbered lists work universally. Code blocks hide structure from markdown-aware parsers. The verdict gate is operator readability AND machine parsability — both must hold.

#### Step 3 — Verdict based on smoke results

- [ ] If **any** smoke fails: include it in `Requirements Verification` as **FAIL** with description + operator observation. **Verdict = REJECTED.** Implementer must `/sdd-fix`.
- [ ] If **all** smokes pass: incorporate as PASS rows in `Requirements Verification` and proceed to APPROVED / APPROVED WITH DEFERRALS based on the rest of the checklist.
- [ ] **Never** mark smoke as "deferral #1 → operator verifies during end-issue". Smoke is gate, not deferral. The Deferrals table is for **post-merge follow-ups** (new Linear issues), not for "the operator hasn't tested yet".

#### Step 4 — Reviewer role boundary

- [ ] Reviewer (this `/sdd-check` run) must NOT run smokes themselves — neither `HUMAN` nor `AGENT` ones. Your job is to *author and classify* the smoke list, then STOP. `HUMAN` smokes are run by the operator; `AGENT` smokes are dispatched by the orchestrator to a separate headless runner. Either way the verdict is `SMOKE TEST REQUIRED` and the result comes back to you / the orchestrator out-of-band.
- [ ] Reviewer must NOT proceed to verdict (APPROVED/REJECTED) without smoke confirmation. For `AGENT` smokes the orchestrator's runner supplies PASS/FAIL/DEFERRAL/BLOCKED-INFRA; for `HUMAN` smokes the operator does. Polling once is enough — wait for the reply, do not assume.
- [ ] Classify honestly: marking a genuinely-visual smoke as `AGENT` to avoid the human gate will produce a false PASS. When unsure, `HUMAN`.

5. **PRODUCE** structured report with **mandatory tables** (PASS/FAIL per criterion):

```markdown
## Verification Report: [ARGUMENTS]

> Review type: Independent session | Same-session (⚠️)

### Requirements Verification

| # | Requirement | Result | Evidence |
|---|-------------|--------|----------|
| RF-01 | [description] | PASS / FAIL | `file:line` or explanation |
| RF-02 | ... | ... | ... |

### Anti-Pattern Verification

| # | Anti-Pattern | Result | Details |
|---|-------------|--------|---------|
| AP-01 | Interfaces 1:1 | PASS / FAIL | X interfaces, Y with 1 impl |
| AP-02 | Disproportionate security | PASS / FAIL | |
| AP-03 | Security theater | PASS / FAIL | |
| AP-04 | Systematic duplication | PASS / FAIL | |
| AP-05 | God objects | PASS / FAIL | Max file: X LOC, Max method: Y LOC |
| AP-06 | Wrong layer | PASS / FAIL | |
| AP-07 | Dead code | PASS / FAIL | |
| AP-08 | Multiple patterns | PASS / FAIL | |

### Build and Tests

| Check | Result | Details |
|-------|--------|---------|
| Build (zero errors) | PASS / FAIL | |
| Build (zero warnings) | PASS / FAIL | X warnings |
| All tests pass | PASS / FAIL | X/Y passed |
| Tests cover spec requirements | PASS / FAIL | |

### Code Metrics

| Metric | Value | Status |
|--------|-------|--------|
| Abstractions total | X | |
| Abstractions with 1 impl | Y | GREEN / RED |
| Max file LOC | X (`file`) | GREEN / YELLOW / RED |
| Max method LOC | X (`file:method`) | GREEN / YELLOW / RED |
| Duplicated code | [list or "none"] | |

### Spec Deviations
- [deviation and impact, or "None"]

### Security
- [verification status]
- [measures with enforcement vs without enforcement]

### Retrospective
- Would I do the same? [Yes/No and why]
- Simpler alternative? [If exists]
- Anti-patterns avoided? [Which and how]

### Deferrals (only if verdict is APPROVED)

> Items that are acceptable to defer to a separate issue. NOT blockers — the code is good enough to merge.
> Each deferral MUST have a Linear issue created by the reviewer during this check.
> Each deferral Linear issue MUST carry the `tech-debt` label — see `SDD_DISCIPLINE.md` §Rule 3 (create the label in the team if missing, then apply during issue creation).

#### Deferral signals — actively look for these (audit-style 2026-05-31)

A pattern observed across the eval cycle (kimi/codex/opus comparisons): opus-class reviewers
**under-report deferrals** because they apply "true trade-off only" too strictly and declare
everything-else as ✅ silently. Codex-class reviewers catch more legitimate follow-ups because
they tolerate "could be extended" framing. Both readings are valid, but legitimate follow-ups
that get silently swallowed turn into rediscovered work months later (more expensive than
opening a small Linear issue now).

**You MUST open a deferral whenever you observe ANY of:**

1. **Any anti-pattern hits a NOTE / yellow zone** (e.g. file 500–600 LOC, method 35-55 LOC,
   abstraction with 2 implementations where 1 of them is trivial). Even if you mark it ✅ NOTE
   in the AP table, open a deferral *to track the trajectory* — yellow becomes red without
   warning, and the eventual split is cheaper if it's already a Linear ticket.
2. **Tests cover this change's happy path but a sibling enum/branch/case in the SAME file
   still lacks coverage.** The reviewer's privileged "I just read the whole file" position
   makes them best-placed to spot it. Operators reading the diff six months later cannot.
3. **A `TODO`/`FIXME`/`HACK` comment in code you touched** that wasn't part of THIS issue's
   scope. Don't fix it — defer it, with the file:line reference.
4. **Pattern reuse opportunity surfaced by the change**: this PR introduced a helper / writer /
   mapper that should logically also replace 1+ duplicated implementations elsewhere, but the
   replacement is out-of-scope here.
5. **Doc comment / CHANGELOG entry / AGENTS.md row that drifted** because of this change but
   is technically separate text-only work.
6. **A spec / plan inversion you discovered** during review — the implementation is correct
   under the operator's *real* intent, but the spec wording is misleading and will trip the
   next reader.

Deferrals from these signals are **net positive** — they cost ~30 seconds to write, they prevent
silent rot, and they make tech-debt visible to the operator (who can then prioritise / dismiss).
Err on the side of opening rather than skipping. Anti-patterns/follow-ups that are TOO MINOR
to be a Linear issue (e.g. one trailing space) should be fixed inline now or ignored, not deferred.

| # | Deferral | Reason for deferring | Linear Issue | Priority |
|---|----------|---------------------|-------------|----------|
| 1 | [description] | [why it's acceptable to defer] | [XX-NNN](url) | P1/P2/P3 |

_If no deferrals, omit this section._ But if you're about to omit it on a >500-LOC change with
a green AP table, **re-read signals 1–6 above** — a clean AP table for a substantial change
usually means signal 1 (yellow zones), 2 (test gaps), or 4 (pattern-reuse) is present and
worth a deferral.

---

**VERDICT: APPROVED / APPROVED WITH DEFERRALS / REJECTED**

- **APPROVED**: All criteria PASS. Ready for `/end-issue`.
- **APPROVED WITH DEFERRALS**: All criteria PASS but some improvements are deferred. Linear issues created for each deferral. Ready for `/end-issue`.
- **REJECTED**: One or more RF-XX or critical AP-XX is FAIL. Implementer must run `/sdd-fix`.

**Corrective actions** (if REJECTED):
1. [action]
```

6. **CREATE LINEAR ISSUES FOR DEFERRALS** (only if verdict is APPROVED WITH DEFERRALS):
   - **MUST use `scripts/linear.py create`** (NOT Linear MCP) — direct GraphQL = zero context bloat:
     ```bash
     python "$ORCH_HOME/scripts/linear.py" create \
       --team DenTherm \
       --title "<deferral title>" \
       --description "<context + link to parent issue + scope>" \
       --priority 3 --estimate 2 \
       --labels tech-debt
     ```
     The script auto-creates the `tech-debt` label in the team if missing (SDD_DISCIPLINE.md §Rule 3) and applies it during creation, then prints the new ID. Capture the ID from stdout.
   - Record the issue ID in the Deferrals table of the report
   - Deferrals are NOT corrections — they are accepted work for a future cycle

7. **DOCUMENTATION MAINTENANCE** (mandatory after approved check):
   - **REMOVE** temporary and irrelevant files
   - **UPDATE** affected documents (AGENTS.md, QUALITY_GATES.md, ANTI_PATTERNS.md, CHANGELOG.md)
   - **VERIFY** consistency between files
   - **CLEAN** completed specs (mark as DONE, remove TODOs)

8. **RECOMMEND** next steps

## Persist Report as File

After producing the report, **WRITE** it as `check-report.md` in the **root of the code repository** (same folder as the .sln/.csproj):

```bash
# Example:
./check-report.md    # root of the code repo where implementation happened
```

This file is the **handoff artifact** between reviewer and implementer:
- **REJECTED**: Implementer reads this file via `/sdd-fix`, applies corrections, then **deletes** it to signal "ready for re-review"
- **APPROVED** or **APPROVED WITH DEFERRALS**: File stays in the repo root. `/end-issue` verifies this file exists with an APPROVED verdict before allowing merge. Deferrals already have Linear issues created.

If a previous `check-report.md` exists, **overwrite** it.

> **Note**: This file should NOT be committed to git. Add `check-report.md` to `.gitignore` if not already there.

## Out of Scope — NEVER DO in this phase

- **NEVER** mark the Linear issue as Done or update its status (that is END-ISSUE step 7)
- **NEVER** push code, create PRs, or merge (that is END-ISSUE)
- **NEVER** close or comment on GitHub Issues
- **NEVER** run `/sdd-end-issue` — that is a separate phase

This phase ONLY produces `check-report.md`. The workflow continues with FIX (if rejected) or END-ISSUE (if approved).

## Output

`check-report.md` written to the code repo root. If APPROVED (with or without deferrals), ready for `/end-issue`. If REJECTED, implementer must run `/sdd-fix` to apply corrections.
