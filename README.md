# SDD Prompts & Standards

Shared prompt files and quality standards for the **Specification-Driven Development (SDD)** workflow used across Densare repositories.

## What is SDD?

SDD is a development methodology where specifications are the source of truth. Each phase of the software development lifecycle has a dedicated prompt that guides AI agents through the process.

## Structure

```
sdd-prompts/
├── ANTI_PATTERNS.md          # Universal anti-patterns (all stacks)
├── aec/                      # AEC area (C#/.NET, Avalonia, desktop)
│   ├── QUALITY_GATES.md
│   └── WORKFLOW.md
├── cloud/                    # Cloud area (Go, Templ, HTMX)
│   ├── QUALITY_GATES.md
│   └── WORKFLOW.md
└── prompts/                  # Universal SDD phase prompts
    ├── request.md
    ├── specify.md
    ├── plan.md
    ├── implement.md
    ├── check.md
    ├── fix.md
    ├── bug.md
    ├── end-issue.md
    ├── verify-end.md
    ├── close.md
    ├── archive.md
    ├── audit.md
    ├── status.md
    └── validate.md
```

## Prompts

| Prompt | Phase | Description |
|--------|-------|-------------|
| `request.md` | REQUEST | Capture user needs in simple language |
| `specify.md` | SPECIFY | Translate request into technical specification |
| `plan.md` | PLAN | Generate implementation plan from spec |
| `implement.md` | IMPLEMENT | Implement code following the plan |
| `check.md` | CHECK | Verify implementation against spec |
| `fix.md` | FIX | Apply corrections from code review |
| `bug.md` | BUG | Investigate bugs and plan fixes |
| `end-issue.md` | END-ISSUE | Finalize issue (commit, PR, merge) |
| `verify-end.md` | VERIFY-END | Verify end-issue completed correctly |
| `close.md` | CLOSE | Verify completion, move to concluidos/ |
| `archive.md` | ARCHIVE | User acceptance testing, move to arquivados/ |
| `audit.md` | AUDIT | Quick triage of possibly implemented requests |
| `status.md` | STATUS | View current state of tasks |
| `validate.md` | VALIDATE | Validate AI-generated spec blocks |

## Standards

| File | Scope | Description |
|------|-------|-------------|
| `ANTI_PATTERNS.md` | Universal | 8 anti-patterns to avoid (lessons from DenStudio) |
| `aec/QUALITY_GATES.md` | AEC | Quality checks per phase (C#/.NET, Clean Architecture) |
| `aec/WORKFLOW.md` | AEC | Full SDD workflow for desktop apps |
| `cloud/QUALITY_GATES.md` | Cloud | Quality checks per phase (Go, Templ, HTMX) |
| `cloud/WORKFLOW.md` | Cloud | Full SDD workflow for SaaS products |

## Usage

Referenced by `.claude/commands/sdd-*.md` and `AGENTS.md` files in Densare repositories via raw GitHub URLs:

```
https://raw.githubusercontent.com/densare/sdd-prompts/master/prompts/<prompt>.md
https://raw.githubusercontent.com/densare/sdd-prompts/master/ANTI_PATTERNS.md
https://raw.githubusercontent.com/densare/sdd-prompts/master/<area>/QUALITY_GATES.md
```

## License

Proprietary. See [LICENSE](LICENSE).
