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
