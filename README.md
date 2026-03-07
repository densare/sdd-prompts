# SDD Prompts

Shared prompt files for the **Specification-Driven Development (SDD)** workflow used across Densare repositories.

## What is SDD?

SDD is a development methodology where specifications are the source of truth. Each phase of the software development lifecycle has a dedicated prompt that guides AI agents through the process.

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

## Usage

These prompts are referenced by `.claude/commands/sdd-*.md` files in Densare repositories via raw GitHub URLs.

## License

Proprietary. See [LICENSE](LICENSE).
