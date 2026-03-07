# SDD Phase: STATUS

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Show the current state of an individual task or global project overview.

## Arguments

| Input | Mode | Example |
|-------|------|---------|
| No arguments | **Global**: all projects | (no args) |
| `<project>` | **Project**: all tasks | see AGENTS.md |
| `<project> <task>` | **Task**: detailed state | see AGENTS.md |

See AGENTS.md for available projects.

## Linear Access — MANDATORY

Issues are tracked in **Linear** (project management tool). **Linear is the source of truth for issue status, NOT STATUS.md files.** You MUST query Linear for every request that has issues.

Methods (in order of preference):
- **MCP tools**: If available (e.g., `mcp__linear__get_issue`, `mcp__linear__list_issues`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

**CRITICAL**: NEVER trust STATUS.md as source of truth for issue progress. STATUS.md is a cache that may be stale. Always query Linear first, then update STATUS.md to match.

## Mode 1: Global (no arguments)

For each project in `projectos/`: count tasks by state, show summary.

## Mode 2: Project (project only)

1. **READ** `projectos/<project>/STATUS.md` (as stale cache — do NOT trust values)
2. **READ** `projectos/<project>/requests/README.md`
3. **LIST** requests in all subfolders
4. For EACH request in `em-implementacao/`: read `issues.md`, **QUERY LINEAR for every issue** to get real status
5. **UPDATE** `projectos/<project>/STATUS.md` with Linear-verified data
6. **REGENERATE** `projectos/<project>/requests/README.md`

## Mode 3: Individual Task (project + ID)

1. **LOCATE** request in `projectos/<project>/requests/`
2. **VERIFY** existence of each file (request.md, spec.md, plan.md, issues.md)
3. If `issues.md` exists: query Linear
4. Check cross-module dependencies

## Task State Determination

```
No request.md or spec.md           -> NOT STARTED
With request.md, without spec.md   -> REQUEST
With spec.md in DRAFT              -> DRAFT
With spec.md SPECIFIED, no plan.md -> SPECIFIED
With plan.md in DRAFT              -> PLANNING
With plan.md PLANNED, no issues    -> PLANNED
With plan.md PLANNED + Linear issues:
  - All Backlog/Todo              -> PLANNED
  - Any In Progress               -> IN_PROGRESS
  - Some Done, others not         -> IN_PROGRESS (partial)
  - All Done                      -> READY TO ARCHIVE
  - All Done or Cancelled         -> READY TO ARCHIVE
```

## Update STATUS.md

```markdown
# <Project> - Task Status

> Automatically updated. Last update: <YYYY-MM-DD HH:MM>

## Summary

| State | Count |
|-------|-------|
| Planned | X |
| In Implementation | X |
| Ready to Archive | X |
| Others | X |

## Tasks

| Task | State | Issues | Progress | Next Step |
|------|-------|--------|----------|-----------|
| <task> | <state> | <issues> | X/Y Done | <command> |
```

## Rules

- DON'T modify code or specs — just reading + STATUS.md + README.md
- Linear is the **source of truth** — STATUS.md is just a cache
- If Linear not accessible: mark **every issue** as "Linear: NOT VERIFIED" and add a warning at the top of STATUS.md
- README.md is ALWAYS regenerated from real file state
- NEVER report an issue as Done/In Progress based solely on STATUS.md — always confirm with Linear
