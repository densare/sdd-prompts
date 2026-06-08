# SDD Discipline

> Universal rules for agents and humans running SDD review phases (`/sdd-check`, `/sdd-end-issue`).
> Single source of truth — repos point here; never duplicate.

Two rules captured from incidents in the densare ecosystem. They apply regardless of stack, repo, or whether you are an AI agent or a human reviewer.

---

## Rule 1 — Review never implements

Review/integration sessions (those running `/sdd-check` and `/sdd-end-issue`) **never write code**, even for "obvious" 1-line fixes or revert-and-redo. Always hand off via:

- Update `check-report.md` to **REJECTED** with fix recommendation, OR
- Create a new Linear issue with full forensic + fix step-by-step + tests.

### Why

The SDD split (plan/implement vs review/integration) exists to **eliminate implementer bias**. If review takes shortcuts to implement "because it's simple", independence collapses and the next check loses rigor — the reviewer becomes accomplice to their own suggestion. Confirmed by operator feedback after the DT-576 incident (2026-05-15): *"tu serves para check e end issue. Só corriges coisas muito simples"*.

### How to apply

- `/sdd-check` reveals a problem → REJECTED + fix proposed in `check-report.md` → handoff. Do not write code.
- `/sdd-end-issue` smoke fails → STOP, create a Linear issue capturing the regression, comment on the issue under review, halt workflow. Do not write code.
- **Exception (trivial-trivial only):** typos in comments / README, formatting that the project's formatter would fix automatically, `.gitignore` entries. Nothing that touches logic, signature, semantics, or tests.
- When in doubt: **it is not trivial** — open the issue.

---

## Rule 2 — Edge cases escalate, never get classified as "small UX window"

During `/sdd-check`, any flow alternative not covered by tests must become one of:

1. **Blocker (REJECTED + require coverage)** — default. The implementer adds the regression test before re-review.
2. **Deferral with Linear issue (P2 or P3)** — when the alternative flow is rare but affects correctness.
3. **Trade-off** — ONLY with an explicit technical reason documented (e.g., IEEE 754 precision outside the regulatory case) AND operator concurrence.

### Anti-pattern to avoid

Classifying an edge case as *"pequena janela visual"* / *"small UX issue"* / *"unlikely in practice"* and proceeding to APPROVED.

**Why it fails:** confirmed by the DT-576 incident — an edge case ("TypeDescription stale after solution change") was accepted as cosmetic; the smoke at `/sdd-end-issue` revealed it was **permanent** (never reloaded in a typical edit session because of a separate latent bug — a `DataChanged` event that never fired) and became immediately visible to the operator. The fix forced a third commit on the same branch and required creating a follow-up issue mid-end-issue.

### How to apply

- New flow discovered during check → **assume worst-case observability**; require a regression test before APPROVED.
- Document the edge case explicitly in the Requirements table as **FAIL (no coverage)**, not PASS-with-asterisk.
- If the fix is trivial: REJECTED with a fix sketch. If complex: APPROVED WITH DEFERRALS + Linear issue **P2** (not P3).
- The Deferrals table only accepts true trade-offs (precision, non-regulatory cases, optimisations). UX/correctness edge cases **become blockers**, not deferrals.

---

## Rule 3 — Every Linear deferral carries `tech-debt` AND inherits its source issue's labels

Any Linear issue you create as a **deferral** from an SDD phase (specify, plan, implement, check, fix, end-issue) **MUST** carry:

1. the `tech-debt` label in its team's workspace, **and**
2. **every label its source issue carries.** A deferral of an `F1`-labelled issue is itself `F1`; of a `Bug` is `Bug`; etc. If the source's scope comes from a **parent epic** rather than its own label (epic children are often unlabelled), apply the **epic's** label too.

No exceptions. The inherited scope labels go *on top of* `tech-debt`, never instead of it.

### Why

Operators rely on the label to filter the orchestrator-generated tech-debt backlog separately from feature work (`backlog-followup.md`, queue triage, prioritisation reviews). A deferral without the label is invisible to that workflow — it pollutes the feature backlog and won't get picked up in tech-debt sweeps.

The **label-inheritance** half (added 2026-06-08) closes the inverse leak: a deferral born from a scoped issue (e.g. an `F1`/release-tagged issue) but tagged only `tech-debt` silently *escapes* that scope — it vanishes from the release/scope view that drives what gets finished. Carrying the source's labels keeps the deferral inside the same scope filter as its origin. Incident: DT-761 (deferred from the F1 epic DT-750's child DT-756) was born `tech-debt`-only and nearly fell out of the F1 close-out set.

Captured 2026-05-30 after the DT-582/DT-662/DT-663 incident where a deferral was opened twice (end-issue agent didn't know the operator had already opened DT-662, opened DT-663 as duplicate). The duplicate revealed that there was no machine-readable signal distinguishing tech-debt from real backlog.

### How to apply

When creating a Linear deferral via MCP (or any other path):

1. **Check** if the label `tech-debt` exists in the issue's team (`workspace.team.labels`).
2. **Create** it if missing — use this canonical form (don't invent variants):
   - **name**: `tech-debt` (lowercase, hyphen, no spaces)
   - **color**: neutral grey (`#6e6e6e` is the canonical choice)
   - **description**: `Divida tecnica / deferral / cleanup — fora do roadmap de fases.`
3. **Read the source issue's labels** (`linear.py get <SOURCE-ISSUE>` → `labels.nodes[].name`). These must be carried onto the deferral. If the source is an epic child with no own labels, read the **parent epic's** labels instead and inherit those.
4. **Apply** `tech-debt` **plus the inherited source labels** to the new issue *during creation* — pass them together via `--labels "tech-debt,<inherited…>"` (`linear.py create`), or the combined label IDs in the `labelIds` field of the MCP create mutation. Preserve any labels the issue already has from templates.
5. **Verify** post-create: the response payload includes the labels list — sanity-check it contains `tech-debt` **and every inherited label**. If any is absent, retry the missing ones as a separate `linear.py label --add "<labels>" <DEFERRAL>` (or label mutation).

This is a hard rule, not a recommendation: an unlabelled deferral is a defect. If the Linear MCP call fails to apply the label, do not proceed — surface the failure in the phase output so the operator can fix it.

---

## Related

- [`ANTI_PATTERNS.md`](./ANTI_PATTERNS.md) — 10 anti-patterns to avoid in code (mandatory reading before any SDD phase). AP-09 (infrastructure-only without entry-point) and AP-10 (storage semantics inconsistency) added in 2026-05 after the DT-547..589 and PLT-205/207/281 chains.
- [`aec/QUALITY_GATES.md`](./aec/QUALITY_GATES.md) / [`cloud/QUALITY_GATES.md`](./cloud/QUALITY_GATES.md) — quality gates by phase.
- [`prompts/check.md`](./prompts/check.md) — the `/sdd-check` prompt itself (this file complements it with operational discipline).

## Provenance

Originally captured as Claude auto-memory after the DT-576 ticket chain (2026-05-15). Migrated to this repo because the rules are stack-agnostic and benefit the whole ecosystem — both AEC (DenTherm, DenStudio) and cloud (densare-platform, densare-admin, 2send, 2snip, densare-sdk-dotnet).
