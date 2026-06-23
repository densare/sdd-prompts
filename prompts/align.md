# SDD Phase: ALIGN (alignment status — code vs reformulated spec)

## Role

You audit **ALIGNMENT**, not progress. The projects were **re-created from scratch (reformulation)**. Much code is **legacy built before the reformulation**. The question is NOT "how many issues are Done" — it is **"does the code that exists match what the reformulated spec asks for?"** Output verdicts: **ALIGNED / REWORK / MISSING**, per capability and per request.

## When to use

During the reformulation phase, to produce the real "alignment status" of a project/epic — distinct from `/sdd-status` (which counts Linear Done) and `/sdd-audit` (per-request DONE/PARTIAL). Use this to know what existing code **serves** the reformulation, what must be **reworked**, and what is **missing**.

## Arguments

`$ARGUMENTS` = `<project> [epic-or-request]`. Examples: `cloud P-LIC`, `cloud plataforma-base P-PAY`, `aec denstudio I0.5`. With no epic/request, do the whole project epic-by-epic.

## CRITICAL rules (non-negotiable — these are why past audits were wrong)

1. **State of code = `origin/main`, fetched. NEVER the local working-tree.** Always `git -C <repo> fetch -q origin` first, then read via `git -C <repo> show origin/main:<path>` and `git -C <repo> ls-tree -r --name-only origin/main <dir>`. Local checkouts may be hundreds of commits stale (this has caused repeated false "not implemented" verdicts). Do **not** `cd` into a stale working tree and grep it.
2. **State of issues = Linear** (source of truth), not local `issues.md`/`STATUS.md`/`em-analise` banners (they rot). Use Linear MCP (`mcp__linear__list_issues`) or `scripts/linear.py`.
3. **"Done" ≠ "aligned".** A legacy issue marked Done in Linear may NOT satisfy the reformulated spec. Count alignment against the spec, never against Done.
4. **Beware the false ✅.** A symbol existing with the right *name* is not alignment — check **shape, place, and contract** (e.g., an interface named `LicenseProvider` that is concrete getters in the wrong package is REWORK, not ALIGNED). Grep-by-symbol lies; read the code.
5. **Distinguish MISSING from BLOCKED-upstream.** If a capability has no code because it depends on another reformulation unit not yet built, say "MISSING (blocked by <X>)", not just "MISSING".
6. **C2 "neutral provider port" capabilities are audited in the CONSUMERS, NOT the platform service.** Provider-abstraction is the app's job, not the service's (`cloud/CLAUDE.md` Provider Abstraction). When a capability is "expose a neutral `XProvider` port (C2)", check the **consumers** (apps + SDK — e.g. `2snip internal/auth/provider.go`, `densare-sdk-dotnet IAuthService`) for the neutral interface; a "C2 port missing" verdict **against the platform service is a FALSE POSITIVE** unless that service has **2+ real backends per cluster** (billing: `mor`/`pt-invoicing`; storage: MinIO/Hetzner/BYO → those DO need an internal port). A single-impl delegation (auth→Zitadel, 1 IdP) needs **no** platform-side port. (This is why the P-AUTH/P-LIC audits false-flagged C2 — they checked the platform; the consumers already had it.)

## Reference (what SHOULD be — assumed correct)

- **`projectos/<project>/FUNCIONALIDADES.md`** — the canonical functional map. Read the target epic's "Deve…" capabilities.
- **`projectos/<project>/ARQUITETURA*.md`** — the v2 architecture constraints (e.g., neutral provider ports, cells, `(tenant,app)` keying) the capabilities must take.
- **The reformulated requests** under `projectos/<project>/requests/**` for that epic — **INCLUDING `em-analise/` (staging)**. A capability may have a request that was planned (has `issues.md`) but **never seeded to Linear** → it is **invisible in a Linear-only view**, and that is exactly the gap to catch (it is how P-AUTH "looked closed" while SVC/C2/headless/(tenant,app) were unbuilt). Cross **FUNCIONALIDADES (capability) ↔ requests (incl. em-analise) ↔ Linear ↔ `origin/main` code** — a coverage check, not just a code-quality review.

## Sizing (this is a "big-picture" 2nd pass — fit the reviewer's context)

This audit reads a coherent chunk holistically. Keep the **material (origin/main code + spec + requests) ≤ ~150–180k tokens** so it fits a reviewer with room to reason (binding reviewer = **kimi 250k**; opus/codex ~200k; gemini ~1M). Before starting, sanity-check the size (`git show` files + specs, chars ÷ ~3.5); **if a big epic overflows (e.g. ~164pts ≈ ~200k+ tokens of code), split it into 2 audits**. Don't fill the window — fidelity drops near the limit.

## Behavior

1. Identify the target epic/request and its capabilities (from FUNCIONALIDADES + requests, **incl. em-analise staging**).
2. `git fetch` the code repo(s); locate the relevant code in `origin/main` (`ls-tree` + `show`).
3. Check Linear for the related issues' real state (context, not the verdict) — and note capabilities/requests with **zero Linear issues** (planned-but-unseeded = a real gap).
4. For **each capability** and **each reformulated request**, assign a verdict:

| Verdict | Meaning |
|---|---|
| ✅ **ALIGNED** | `origin/main` implements it in the v2 form the spec asks (cite file/symbol). |
| 🟡 **REWORK** | Legacy code does something similar but **not in v2 form** (wrong shape/place/contract/keying). Say what exists + what must change. |
| ❌ **MISSING** | No code satisfies it. Add "(blocked by <X>)" if upstream-blocked. |
| ❔ **UNSURE** | Cannot decide from code alone (possible deliberate decision) — flag for the owner. |

## Output (European Portuguese — planning repo language)

- **Table A — capabilities** (`Capacidade | Veredito | Evidência em origin/main | Gap p/ v2`).
- **Table B — reformulated requests** (`Request | Veredito | Evidência | Gap`).
- **Quantitative summary** (#✅ / #🟡 / #❌ / #❔).
- **Epic verdict**: is it aligned with the reformulation, or does "Done" hide legacy-to-realign? Call out structural changes that are NOT wrappers (data-model rekeying, greenfield subsystems) so they are not under-estimated as "adapts".
- **Dependency notes**: what blocks what (cross-epic).
- **❔ for the owner**: explicit questions only the owner can answer (gap vs deliberate decision).
- **Optionally update** `projectos/<project>/STATUS.md` (or `STATUS_PLANNING.md`) with the alignment snapshot — but mark it ALIGNMENT, never conflate with Done counts.

## Rules

- DO NOT write code. DO NOT edit the code repo. This is read-only audit + planning notes.
- DO NOT trust local working-trees or `em-analise` banners. Linear + `origin/main` only.
- For high-stakes epics, a second adversarial pass ("is there really no other code path for this, in another product/package?") strengthens MISSING verdicts.
