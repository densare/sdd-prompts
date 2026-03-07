# SDD Phase: AUDIT

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Perform a quick triage of a request, checking if it's already (fully or partially) implemented in code.

## Required Context

Read or have access to:
- `AGENTS.md` — project rules, repositories, module classification
- The indicated request

## Arguments

Format: `<project> <request-id>` — see AGENTS.md for available projects.

If only project provided: list available requests and their audit status.

## Context

Many requests were written for features that may already exist in code. Instead of following the full SDD pipeline, we first do a quick audit to decide the correct path.

## Linear Access

Issues are tracked in **Linear** (project management tool). To search for existing issues:
- **MCP tools**: If available (e.g., `mcp__linear__list_issues`, `mcp__linear__get_issue`)
- **GraphQL API**: `https://api.linear.app/graphql` — see [Linear API docs](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)
- **CLI**: `linear-cli` if installed

## Locate Request

Search in `projectos/<project>/requests/` (in order):
1. `aprovados/<ID>-*.md`
2. `em-analise/<ID>-*.md`

## Behavior

### Mode 1: Without request-id (list)
1. Read `projectos/<project>/requests/README.md`
2. Read `projectos/<project>/AUDITORIA.md` (if exists)
3. Show summary by epic

### Mode 2: With request-id (audit)
1. **READ** the complete request
2. **CLASSIFY** using the project's module classification (from AGENTS.md)
3. **IDENTIFY** correct repository
4. **CHECK LINEAR** (the project management tool) — search for existing issues covering this request
5. **INVESTIGATE** existing code
6. **CLASSIFY** the state:

| State | Meaning | Next Action |
|-------|---------|-------------|
| **DONE** | 100% implemented as requested | Close |
| **PARTIAL** | Part implemented, missing aspects | Create delta-request |
| **DIVERGENT** | Implemented but different | Decide: adjust code OR request |
| **NOT STARTED** | Nothing implemented | Normal SDD pipeline |

7. **PRODUCE** report:

```markdown
## Audit: <ID> - <Title>

**Classification**: [module]
**Repository**: <repository>
**State**: <DONE/PARTIAL/DIVERGENT/NOT STARTED>

### Related Linear Issues
| Issue | Title | Status | Impact |
|-------|-------|--------|--------|
| <ID> | <title> | Done/... | <impact> |

### Evidence (Code)
- `<file>:<line>` — <what it does>

### Request Coverage
| Request Point | State | Evidence |
|---------------|-------|----------|
| <point> | Yes/No/Partial | <file or N/A> |

### Decision
<DONE/PARTIAL/DIVERGENT/NOT STARTED with next action>
```

8. **UPDATE** `projectos/<project>/AUDITORIA.md`

## Rules

- DON'T create specs, plans or code — this is just triage
- DON'T modify the request — only classify
- DON'T spend more than ~15 min per request — it's meant to be quick
- Search for equivalent functionality, not just exact names

## Integration with SDD

```
                                 +--> DONE (close)
                                 |
Request --> AUDIT ---------------+--> PARTIAL --> SPECIFY (delta)
                                 |
                                 +--> DIVERGENT --> decide with user
                                 |
                                 +--> NOT STARTED --> SPECIFY
```

## Output

Audit report + updated AUDITORIA.md. Recommend next steps.
