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

Issues are tracked in **Linear** (project management tool). To query issues:
- **MCP tools**: If available (e.g., `mcp__linear__get_issue`, `mcp__linear__list_issues`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

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
>
> **Anti-pattern observed in practice:** deferring smoke as "deferral #1 → part of end-issue checklist". Result: smoke fails during end-issue → STOP/REJECT/restart cycle (e.g., DT-576, DT-577). Correct workflow: smoke during check, end-issue mechanical.

#### Step 1 — Locate the smoke list

- [ ] Read `plan.md` §"Planned tests" → "Manual smoke tests" / "Smoke manual" list.
- [ ] If the list is **empty** AND the issue body has observable acceptance criteria (e.g., "lista mostra X", "painel abre Y", "edit field → value persists"):
  - This is a **plan deficiency**. Flag in `Spec Deviations`.
  - **STOP** and derive smokes from the acceptance criteria yourself, then proceed to Step 2 (do NOT skip smoke gate just because plan is incomplete).
- [ ] If both empty (no smokes planned AND no observable acceptance criteria — e.g., pure refactor with unit-test coverage): document as "Smoke: N/A (no observable behavior)" in the report and skip to anti-patterns.

#### Step 2 — Present the smoke list to the operator as numbered step-by-step recipes (one per smoke)

- [ ] **STOP all further verification work.** Before producing the check report, ask the operator to run the smokes. Present each smoke as a **separate block with a bold heading and a vertical numbered list** (one step per line). Do NOT use a markdown table with line-break trickery (`<br>` inside cells does not render line breaks in most terminal markdown renderers — steps collapse into a paragraph).

##### Format template (mandatory)

```markdown
Run the following smoke tests in the running app and reply with PASS/FAIL per smoke + observations.
Each block is a separate scenario — test them individually, not as a batch.

---

**Smoke 1 — <field or flow name>**

1. <setup action, e.g. "Importar `process1.dtz`">
2. <navigate, e.g. "Abrir painel Envolvente">
3. <interact, e.g. "Duplo-clicar uma parede">
4. <change, e.g. "Tab **Geometria** → dropdown Orientation → escolher **Norte**">
5. <commit, e.g. "Clicar **OK**">
6. **Deves ver** <observable outcome, e.g. "`Az: Norte` na linha da parede">

---

**Smoke 2 — <next field/flow>**

1. ...

---
```

##### Authoring rules

- **Numbered list** (`1.`, `2.`, ...) — one step per line. Renders correctly in terminals and browsers without `<br>` hacks.
- Use `---` horizontal rule between smokes for visual separation.
- Each step is an **action** (open / click / type / select) **OR** the final **expected observation** (always prefixed with `**Deves ver**` or equivalent in the project's language).
- The expected observation is **the last numbered step**, not a separate paragraph.
- Steps must be **concrete**: name the file, tab, control, value. Avoid generic phrasings ("edit the field") — say "Tab Geometria → dropdown Orientation → escolher Norte".
- Each smoke MUST be **per-field or per-flow**, not generic. Generic smokes ("edit element + OK") can mask field-specific gaps. If the change touches 4 fields, present 4 smokes.
- Include the test data context in step 1 of the first smoke (e.g., "Importar `process1.dtz`" or "Abrir projecto com PEN + ENU"). Subsequent smokes can assume the same project is open.
- End the message with a single line asking for the response format: e.g. `Responde **PASS/FAIL** por smoke + observações. Verdict em hold.`

##### Why no table

Markdown tables collapse multi-line cells in most renderers (terminal CLIs, GitHub markdown viewer with default CSS, plain-text logs). Even when `<br>` works in HTML output, it doesn't in plain markdown rendering. Numbered lists work universally. The verdict gate is operator readability — if the operator can't follow the steps line-by-line, the smoke is harder to execute correctly.

#### Step 3 — Verdict based on smoke results

- [ ] If **any** smoke fails: include it in `Requirements Verification` as **FAIL** with description + operator observation. **Verdict = REJECTED.** Implementer must `/sdd-fix`.
- [ ] If **all** smokes pass: incorporate as PASS rows in `Requirements Verification` and proceed to APPROVED / APPROVED WITH DEFERRALS based on the rest of the checklist.
- [ ] **Never** mark smoke as "deferral #1 → operator verifies during end-issue". Smoke is gate, not deferral. The Deferrals table is for **post-merge follow-ups** (new Linear issues), not for "the operator hasn't tested yet".

#### Step 4 — Reviewer role boundary

- [ ] Reviewer must NOT run smokes themselves (no UI/runtime access). Operator is the source of truth for "did it actually work in the running app".
- [ ] Reviewer must NOT proceed to verdict (APPROVED/REJECTED) without smoke confirmation. Polling once is enough — wait for operator reply, do not assume.

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

| # | Deferral | Reason for deferring | Linear Issue | Priority |
|---|----------|---------------------|-------------|----------|
| 1 | [description] | [why it's acceptable to defer] | [XX-NNN](url) | P1/P2/P3 |

_If no deferrals, omit this section._

---

**VERDICT: APPROVED / APPROVED WITH DEFERRALS / REJECTED**

- **APPROVED**: All criteria PASS. Ready for `/end-issue`.
- **APPROVED WITH DEFERRALS**: All criteria PASS but some improvements are deferred. Linear issues created for each deferral. Ready for `/end-issue`.
- **REJECTED**: One or more RF-XX or critical AP-XX is FAIL. Implementer must run `/sdd-fix`.

**Corrective actions** (if REJECTED):
1. [action]
```

6. **CREATE LINEAR ISSUES FOR DEFERRALS** (only if verdict is APPROVED WITH DEFERRALS):
   - For each deferral, create a Linear issue with: title, description, priority, link to original issue
   - **Apply the `tech-debt` label** (mandatory — see `SDD_DISCIPLINE.md` §Rule 3). Create the label in the team first if missing.
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
