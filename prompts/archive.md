# SDD Phase: ARCHIVE (User Acceptance Testing)

> Universal prompt — language-agnostic, works with any AI tool and any tech stack.

## Role

Generate a manual acceptance test plan for the user to validate the implementation of a completed request. Guide the user through step-by-step verification of all functionality.

## Required Context

Read or have access to:
- `AGENTS.md` — project rules, repositories

## Arguments

Format: `<project> <ID>` — see AGENTS.md for available projects.

## Context

This is the **user acceptance phase** — the final quality gate before a request is fully closed. After `/sdd-close` moves a task to `concluidos/`, this command generates a hands-on test plan for the user to manually validate that the implementation matches the original request.

**Key principle**: The user (Product Owner) is the final judge. The AI prepares the test plan; the user executes it and reports the verdict.

## Locate Task

Search in: `projectos/<project>/requests/concluidos/<ID>-*/`

Must contain: `request.md`, `spec.md`
May contain: `plan.md`, `issues.md`, `close-report.md`

## Behavior

### 1. Read All Artifacts

1. **READ** complete `request.md` — understand what the user originally asked for
2. **READ** complete `spec.md` — understand technical requirements (RF-XX) and acceptance criteria
3. **READ** `plan.md` if exists — understand implementation approach
4. **READ** `close-report.md` if exists — understand what was verified at close time

### 2. Generate Acceptance Summary

Write a plain-language summary (NOT technical) of what was implemented:
- **What it does**: 2-3 sentences in simple language (from request.md perspective)
- **Where to find it**: Which screen/panel/menu in the application
- **Visual expectations**: What the user should see (UI elements, layout, behavior)

### 3. Generate Test Scenarios

For EACH functional requirement (RF-XX) in spec.md, create a test scenario:

```
### Cenario <N>: <description>

**Requisito**: RF-XX — <requirement title>

**Pre-condicoes**: <what must be in place before testing>

**Passos**:
1. <step 1 — action the user performs>
2. <step 2 — next action>
3. ...

**Resultado esperado**: <what should happen / what the user should see>

**Resultado**: [ ] OK  [ ] NOK

**Notas/Defeitos**: _(preencher se NOK)_
```

Rules for test scenarios:
- Cover ALL options/variations mentioned in the request
- Use concrete, realistic example data (not "insert value X")
- Describe visual expectations ("deve aparecer um painel com...", "o botao muda para...")
- Include positive cases AND key negative/edge cases from spec.md
- Order scenarios from basic functionality to advanced features
- Group related scenarios logically

### 4. Generate Quick-Start Example

Create ONE complete end-to-end walkthrough that exercises the main functionality:

```
### Exemplo Completo (Passo-a-Passo)

Este exemplo demonstra o fluxo completo da funcionalidade:

1. Abrir o DenStudio/DenTherm/...
2. ...
3. ...
N. Verificar que o resultado e <esperado>
```

This should be a realistic usage scenario, not a synthetic test case.

### 5. Write Accept Report

Generate `accept-report.md` in the task folder with this structure:

```markdown
# Relatorio de Aceitacao: <ID> - <name>

**Projecto**: <project>
**Data**: <YYYY-MM-DD>
**Fase**: Aceitacao pelo utilizador (UAT)

---

## Sumario

<plain-language summary of what was implemented>

### Onde Encontrar

<where in the application to find this feature>

### O Que Deve Ver

<visual description of expected UI/behavior>

---

## Exemplo Completo

<full walkthrough>

---

## Cenarios de Teste

<all test scenarios>

---

## Resumo

| Cenario | Requisito | Resultado |
|---------|-----------|-----------|
| 1. <desc> | RF-01 | [ ] OK / [ ] NOK |
| 2. <desc> | RF-02 | [ ] OK / [ ] NOK |
| ... | ... | ... |

---

## Veredicto

- [ ] **ACEITE** — Tudo funciona conforme esperado
- [ ] **ACEITE COM RESERVAS** — Funciona mas com defeitos menores (listar abaixo)
- [ ] **REJEITADO** — Defeitos criticos que impedem utilizacao (listar abaixo)

### Defeitos Encontrados

| # | Cenario | Descricao | Severidade |
|---|---------|-----------|------------|
| 1 | | | Critico / Maior / Menor |

### Notas do Utilizador

_(espaco para observacoes livres)_

---

*Gerado por /sdd-archive em <date>. Preenchido pelo utilizador.*
```

### 6. Present to User

After generating the report:
1. **SHOW** the summary section to the user
2. **EXPLAIN** what they need to do: open the application, follow the scenarios, mark OK/NOK
3. **ASK** if they want to proceed with testing now or review the plan first

### 7. Collect Results

After the user completes testing (may happen in same or different session):

**If ACEITE**:
1. Update verdict section in `accept-report.md`
2. Move folder: `concluidos/<ID>-<name>/` -> `arquivados/<ID>-<name>/`
3. Update `requests/README.md` counters

**If ACEITE COM RESERVAS**:
1. Update verdict and defects in `accept-report.md`
2. Create Linear issues for each defect (severity: Minor)
3. Move to `arquivados/` — defects tracked as separate issues
4. Update `requests/README.md` counters

**If REJEITADO**:
1. Update verdict and defects in `accept-report.md`
2. Create Linear issues for each critical defect
3. Do NOT move — stays in `concluidos/` until defects fixed
4. After fixes: user re-runs `/sdd-archive` to re-test

## Rules

- DON'T use technical jargon in the acceptance summary or test steps — this is for the user
- DON'T modify code — this is a testing/validation phase
- DON'T skip any RF-XX from the spec — every requirement gets a test scenario
- DO use realistic example data (Portuguese names, real addresses, typical building data for AEC)
- DO include screenshots references if the spec mentions specific UI elements
- DO make test steps atomic — one action per step, clear expected result
- DO consider the user may not be a developer — instructions must be clear for a Product Owner

## Output

Archive report (`accept-report.md`) in the task folder. Interactive session with user for verdict collection. If accepted, task moves to `arquivados/`.
