# SDD Phase: CLOSE

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Verify that an SDD task was successfully completed and close it (move to `concluidos/`).

## Required Context

Read or have access to:
- `AGENTS.md` — project rules, repositories

## Arguments

Format: `<project> <ID>` — see AGENTS.md for available projects.

## Context

This is the closure of the planning cycle. After all issues of a task are Done, this command does a completion mini-audit and moves the task to `concluidos/`.

## Linear Access

Issues are tracked in **Linear**. **Use `scripts/linear.py` for ALL Linear ops:**

```bash
python "$ORCH_HOME/scripts/linear.py" get DT-NNN
python "$ORCH_HOME/scripts/linear.py" list --team DenTherm --open-only --limit 50 [--json]
python "$ORCH_HOME/scripts/linear.py" update DT-NNN --state Done --comment "..."
```

`$ORCH_HOME` is exported by the orchestrator; API key auto-discovered from `<sandbox>/.linear.env`.

**DO NOT use Linear MCP** (`mcp__linear__*`). The MCP was detached from sandbox `.mcp.json` on 2026-06-01 — it was consuming ~50% of session context. Calling `mcp__linear__*` will fail.

## Locate Task

Search in: `projectos/<project>/requests/em-implementacao/<ID>-*/`

Must contain: `request.md`, `spec.md`, `plan.md`, `issues.md`

## Behavior

### 1. Read Artifacts
1. **READ** complete `request.md`, `spec.md`, `plan.md`
2. **READ** `issues.md`
3. **IDENTIFY** all referenced Linear issues

### 2. Verify Issues in Linear
For EACH issue: query status. ALL must be Done (or Cancelled with justification).
If any NOT Done: STOP and list pending issues.

### 3. Completion Mini-Audit
1. **Every `[bloqueia-close]` CA verified by its Prova?** — for each Acceptance Criterion in the spec, confirm its `Prova:` was executed (test green / oracle diff clean / benchmark met / smoke PASS in `/sdd-check`). **This is the gate that kills "Done ≠ funciona".** A blocking CA without an executed Prova → REJECTED.
2. **Spec requirements covered?** — every RF maps to a verified CA (search for evidence in code)
3. **Tests exist?** — search for test files from plan.md
4. **PRs merged?** — verify on GitHub

### 4. Produce Report

```markdown
## Close Report: <task>

**Project**: <project>
**Date**: <YYYY-MM-DD>

### Linear Issues
| Issue | Title | Status | PR |
|-------|-------|--------|----|
| <ID> | <title> | Done/Cancelled | merged/N/A |

### Acceptance Criteria (verified by Prova)
| CA | Bloqueia-close? | Prova | Verified |
|----|-----------------|-------|----------|
| CA-01 | yes | <test / oracle / benchmark / smoke> | PASS / FAIL / N/A |

### Mini-Audit
| Requirement | CA | Evidence | Status |
|-------------|----|----------|--------|
| RF-01 | CA-01 | <file:line or N/A> | OK / Not found |

### Tests
| Test File | Exists? |
|-----------|---------|
| <path> | Yes/No |

### Verdict
APPROVED FOR CLOSE / REJECTED (with reasons)
```

### 5. Decision

**If APPROVED**:
1. Move folder: `em-implementacao/<ID>-<name>/` -> `concluidos/<ID>-<name>/`
2. Add `close-report.md` to destination
3. Update `requests/README.md`
4. Update `AUDITORIA.md` if exists

**If REJECTED**: List reasons, suggest corrective actions, don't move.

## Rules

- DON'T modify code
- DON'T run builds or tests (CHECK and END-ISSUE already did that)
- If code repo not accessible: mark as "Not verifiable" but continue
- If issue Cancelled: accept IF justified in Linear
- **REJECT if any `[bloqueia-close]` CA has no executed Prova** — a CA silently relaxed/removed after approval (e.g. a loosened threshold) requires an owner-approved note in the request; without it, REJECT.

## Output

Close report. If approved: files moved to `concluidos/`. If rejected: necessary actions.
