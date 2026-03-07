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

Issues are tracked in **Linear** (project management tool). To query issue status:
- **MCP tools**: If available (e.g., `mcp__linear__get_issue`, `mcp__linear__list_issues`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

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
1. **Spec requirements covered?** — search for evidence in code
2. **Tests exist?** — search for test files from plan.md
3. **PRs merged?** — verify on GitHub

### 4. Produce Report

```markdown
## Close Report: <task>

**Project**: <project>
**Date**: <YYYY-MM-DD>

### Linear Issues
| Issue | Title | Status | PR |
|-------|-------|--------|----|
| <ID> | <title> | Done/Cancelled | merged/N/A |

### Mini-Audit
| Requirement | Evidence | Status |
|-------------|----------|--------|
| RF-01 | <file:line or N/A> | OK / Not found |

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

## Output

Close report. If approved: files moved to `concluidos/`. If rejected: necessary actions.
