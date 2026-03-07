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
8. **QUALITY GATE** — Apply SPECIFY checklist:
   - Does task do ONE thing?
   - Does similar code exist? (AP-04)
   - Is there premature generalization? (AP-01)
   - Doesn't mix multiple responsibilities?
   - Is security proportional to data type? (AP-02)
   - Cross-module dependencies identified and documented?
9. **CLARIFY** with user if necessary
10. **ASK** for confirmation to mark as SPECIFIED

## Rules

- DO NOT discuss implementation (that's for PLAN phase)
- Focus ONLY on expected behavior
- ALERT if spec mixes multiple responsibilities
- Keep `request.md` original UNCHANGED
- If task touches 2+ repositories: MANDATORY to split into sub-tasks
- If task depends on non-existent functionality in another module: ALERT

## Output

### Single Repository Task
Folder `<ID>-<name>/` created with `request.md` + `spec.md` (state DRAFT or SPECIFIED).

### Cross-Module Task (2+ Repositories)
Files created inside the folder:
- `spec.md` — index spec that lists sub-tasks and dependencies
- One spec per module (e.g., `spec-core.md`, `spec-app.md`)

Inform implementation order: shared/core modules first, then dependent modules.
