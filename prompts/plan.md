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

Format: `<project> <ID>` — see AGENTS.md for available projects.

## Linear Access

Issues are tracked in **Linear** (project management tool). To create/query issues:
- **MCP tools**: If available (e.g., `mcp__linear__create_issue`, `mcp__linear__list_issues`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

## Locate the Spec

Search for `spec.md` in `projectos/<project>/requests/` (in order):
1. `aprovados/<ID>-*/spec.md`
2. `em-analise/<ID>-*/spec.md`
3. `em-implementacao/<ID>-*/spec.md`
4. If not found, inform that spec must be created first

## Pre-conditions

- Verify that `spec.md` exists and has state SPECIFIED (approved)
- If state DRAFT, inform that spec needs approval first
- If cross-module dependencies exist: check they are PLANNED or IMPLEMENTED

## Behavior

0. **AUDIT** (mandatory step before planning):
   - For each point in the request/spec, check if it already exists in code
   - Classify: DONE / PARTIAL / DIVERGENT / NOT STARTED
   - If **DONE**: STOP — feature already exists
   - If **PARTIAL**: Adjust scope to cover ONLY what's missing
   - If **DIVERGENT**: STOP — present divergences to user
   - Record audit result at the top of plan.md

1. **READ** `sdd/ANTI_PATTERNS.md` — keep the anti-patterns in mind
2. **READ** complete spec.md
3. **VALIDATE CLASSIFICATION**: Confirm target repository
4. **INVESTIGATE EXISTING CODE** (Search Before Create):
   - For each requirement, search source code for similar components
   - Decide by order: MODIFY > EXTEND > GENERALIZE > CREATE
   - **RED FLAG**: If plan has more NEW files than MODIFIED files -> STOP and review
5. **CHECK CROSS-MODULE DEPENDENCIES**:
   - For each dependency NOT_EXISTS or DRAFT: BLOCK
   - For each dependency SPECIFIED: WARN
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
8. **ESTIMATE** using Fibonacci scale:
   - 1 = ~30min | 2 = ~1h | 3 = ~2h | 5 = ~4h | 8 = ~8h
   - >= 13 points: MANDATORY to split
9. **CREATE** `plan.md` with:
   - Target repository
   - Architecture decisions (with justification)
   - Files to create/modify (with layer)
   - Existing packages/modules to reuse
   - Ordered steps (following project's layer order from AGENTS.md)
   - **Dependency graph with parallelization** — for each step, identify what it depends on and what can run in parallel (enables simultaneous work on repos A/B)
   - Planned tests
   - Anti-patterns verified section
   - **Risks and rollback plan** — identify implementation risks with probability/impact/mitigation, and document how to revert if the implementation fails (branch strategy, critical files affected, data reversibility)
   - Cross-module dependencies
   - Total estimate in points
10. **ASK** for confirmation to mark as PLANNED
11. **CREATE LINEAR ISSUE(S)** (issues are tracked in Linear — the project management tool):
    - Title, description, estimate, labels
    - Record issue ID (e.g., DT-48, DS-366) in plan.md
12. **CREATE `issues.md`** with table of Linear issues and implementation order
13. **MOVE** request folder to `em-implementacao/`
14. **UPDATE** `projectos/<project>/requests/README.md`

## Planning vs Implementation Separation

> **This repository is for PLANNING.** Implementation is done in the code repository.

The flow here ends with creating the Linear issue:
```
AUDIT -> SPECIFY -> PLAN -> Linear issue -> END (handoff)
```

## Rules

- DO NOT write code — only plan and document
- **SEARCH BEFORE CREATE** (AP-04)
- **BLOCK** duplicating existing code (AP-04)
- **BLOCK** abstraction with 1 implementation without justification (AP-01)
- **BLOCK** planned file > 600 LOC (AP-05)
- >= 13 points: MANDATORY to split

## issues.md Template

```markdown
# Issues: <ID> - <Title>

## Linear Issues

| # | Issue | Title | Repository |
|---|-------|-------|------------|
| 1 | [XX-NNN](linear-url) | <title> | <repo> |

## Implementation Order

1. **XX-NNN** — <brief description>
```

## Output

Folder in `em-implementacao/` with `request.md` + `spec.md` + `plan.md` + `issues.md`. Linear issues created.
