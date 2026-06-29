# Spec: [TASK-NAME]

> **Estado**: DRAFT | SPECIFIED
> **Classificacao**: Cloud: [PLATFORM] | [APP] | [ADMIN] | [SDK]  ·  AEC: [CORE] | [THERMAL] | [SIM]
> **Repositorio**: Cloud: densare-platform | densare-apps | densare-admin | densare-sdk-dotnet  ·  AEC: denstudio-core | dentherm | densim | denstudio | dendraw
> **Regulado**: sim | nao  (herda do request; `sim` obriga CA de nao-regressao V&V — ver Requisitos Funcionais)
> **Criado**: [DATA]

---

## Resumo

[Descricao breve em 1-2 frases do que esta funcionalidade faz]

---

## Contexto

### Problema
[Qual problema esta funcionalidade resolve?]

### Utilizadores Afetados
[Quem vai usar: utilizadores finais, outros servicos, admin, apps AEC?]

### Valor de Negocio
[Porque e importante implementar isto?]

---

## Requisitos Funcionais

> **RF ↔ CA bidirecional (obrigatorio)**: cada RF referencia o(s) CA do request que satisfaz (`CA-NN`), e cada CA do request mapeia para um RF. **Nenhum RF sem CA** (= requisito nao-testavel) e **nenhum CA orfao**. Cada criterio carrega uma linha **Prova** — o mecanismo que o demonstra; e isso que `/sdd-check` executa e `/sdd-close` lista. **1 CA = 1 obrigacao.**

### RF-01: [Nome do Requisito]
**Descricao**: [O que o sistema deve fazer]
**Satisfaz**: CA-01 [, CA-NN do request]

**Criterios de Aceitacao**:
- [ ] **CA-01** — Dado [contexto], quando [acao], entao [resultado mensuravel]
  **Prova**: [teste automatico | oraculo de regressao | benchmark | smoke manual] · [fixture/dados] · [onde se observa]

### RF-02: [Nome do Requisito]
**Descricao**: [O que o sistema deve fazer]
**Satisfaz**: CA-NN

**Criterios de Aceitacao**:
- [ ] **CA-NN** — Dado ..., quando ..., entao ...
  **Prova**: ...

> **Calculo regulado** (resultado com valor legal): incluir um **CA de nao-regressao** — "toda a suite de oraculos mantem-se dentro da tolerancia aprovada; qualquer diff exige aprovacao do dono" (Prova = runner de regressao/legacy em CI, label `validate-against-legacy`). Oraculos por ID/versao estaveis; **tolerancia zero** no veredicto que cruza um limite legal; comparar tambem **parcelas intermedias**, nao so o total (erros que se compensam batem o total e escondem o bug).

---

## Entry-Points (AP-09)

> **Obrigatorio para features observaveis pelo utilizador / cliente externo.**
> Se a feature e puramente interna (helper, refactor, package partilhado sem observabilidade externa): escrever "N/A — feature nao observavel externamente" e justificar.

Listar TODOS os caminhos que tornam esta feature acessivel em runtime. Sem entry-point declarado, a feature e invisivel ao utilizador apesar de "implementada" — vira issue nova depois do merge.

| Entry-Point | Tipo | Localizacao | Estado actual |
|-------------|------|-------------|---------------|
| [ex: Menu "Relatorios -> Caderno"] | Menu | `DenThermUIProvider.Menus.cs:194` | A criar / A modificar / Ja existe |
| [ex: Nav node "Envolvente -> Pontes Termicas"] | NavTree | `Navigation.cs` | A criar |
| [ex: Keybinding Ctrl+R] | Atalho | `MainWindow.axaml` | A criar |
| [ex: POST /v1/links] | HTTP route | `cmd/2snip/main.go` router | A criar |
| [ex: hx-get="/dashboard/widget"] | HTMX partial | `templates/pages/dashboard.templ` | A criar |

**Smoke navegavel mandatorio**: descrever em 1 frase o caminho que valida acesso a partir de estado inicial (app fresca / deploy limpo). Esta frase vai virar smoke obrigatorio em `/sdd-check`.

> Exemplo: "Abrir app, click menu Relatorios -> Caderno -> tab Folha A renderiza com dados do processo aberto."

---

## Storage Semantics (AP-10)

> **Obrigatorio para features que persistem state.** Se a feature nao persiste nada (puramente computational, view-only): escrever "N/A — sem persistencia" e saltar.

Para CADA campo persistido tocado por esta feature, declarar a forma canonica armazenada e as conversoes em cada path. Forma canonica divergente entre paths e a causa principal de bugs em cadeia (DT-572..578).

| Campo | Forma canonica armazenada | Write paths e conversao | Read path |
|-------|---------------------------|-------------------------|-----------|
| [ex: U-value opaco] | aggravated (× 1.35 ja aplicado) | Import: aggrava antes de gravar. Mass-update: via OpaqueElementUValueService. Dialog edit: chama AggravateU() antes de Save. | Nenhuma — ler como esta |
| [ex: Name elemento] | display synthesized | Import: synthesizer.Resolve(code). Dialog: read-only (gerado). | Nenhuma |

**Helper centralizado**: se 2+ paths escrevem no mesmo campo, deve existir UM helper que aplica a conversao canonical. Nomear de forma que torne a semantics visivel (`ApplyEnvelopeItemToOpaqueElement_storesAggregatedU`, ou comentario unico no helper).

**Mudanca de semantics canonical**: se esta feature ALTERA a forma canonical existente de um campo: ALERTA explicito + plano de migracao (script de migration, backfill, deprecation path).

---

## Requisitos Nao-Funcionais

### RNF-01: Performance
- [Tempo de resposta esperado (ex: < 100ms para auth)]
- [Throughput esperado (ex: 1000 req/s para keep-alive)]

### RNF-02: Seguranca
- [Requisitos de autenticacao/autorizacao]
- [Encriptacao, rate limiting, input validation]

### RNF-03: Disponibilidade
- [Uptime esperado]
- [Comportamento em caso de falha]

### RNF-04: Recursos
- [Limites de memoria (VPS 4GB)]
- [Limites de disco]

---

## API (se aplicavel)

### Endpoints

| Metodo | Path | Auth | Descricao |
|--------|------|------|-----------|
| POST | `/v1/...` | JWT / Publico | [Descricao] |
| GET | `/v1/...` | JWT | [Descricao] |

### Request/Response (exemplos)

```json
// POST /v1/...
// Request
{
  "field": "value"
}

// Response 200
{
  "field": "value"
}
```

---

## Edge Cases

| # | Cenario | Comportamento Esperado |
|---|---------|------------------------|
| EC-01 | [Cenario limite] | [Como o sistema deve reagir] |
| EC-02 | [Erro de input] | [Mensagem ou comportamento] |
| EC-03 | [Servico dependente indisponivel] | [Fallback ou erro] |

### Edge Cases Obrigatorios (features com persistencia) — AP-10

> Se a feature persiste state, estes 4 edge cases sao obrigatorios. Remover apenas com justificacao explicita ("nao aplicavel porque..."). Eliminam por construcao a cadeia DT-572..578.

| # | Cenario | Comportamento Esperado |
|---|---------|------------------------|
| EC-RT1 | **No-op edit**: abrir dialog/form da entidade + OK sem alterar campo nenhum | ZERO mudancas em storage. Todos os campos preservados byte-a-byte. |
| EC-RT2 | **Round-trip**: save -> reabrir projecto -> save sem alteracoes | Output idempotente. Diff = vazio. |
| EC-RT3 | **Import vs novo**: importar legacy + criar novo do zero da app | Mesma estrutura armazenada para o mesmo input semantico. |
| EC-RT4 | **Mass-update sobre item importado**: alterar campo X em N items via mass-update | Forma canonical de campos NAO tocados (ex: U-value) preservada do path de import. |

---

## Dependencias

### Dependencias Tecnicas
- [ ] [Servico X deve existir (ex: PostgreSQL, Redis)]
- [ ] [Package Y deve estar disponivel (ex: pkg/middleware)]

### Dependencias de Outras Tasks
- [ ] [task-name] deve estar completa

### Dependencias Cross-Module

> Preencher se esta task precisa de funcionalidade de outro modulo/repositorio.
> Se nao ha dependencias cross-module, escrever "Nenhuma".

| Dependencia | Modulo | Estado | Spec |
|-------------|--------|--------|------|
| [endpoint/funcao/modelo necessario] | [PLATFORM]/[APP]/[ADMIN]/[SDK] | [NAO_EXISTE/DRAFT/SPECIFIED/PLANNED/IMPLEMENTED] | [path para spec ou "a criar"] |

**Bloqueado por**: [specs que devem estar PLANNED/IMPLEMENTED antes desta]
**Bloqueia**: [specs que dependem desta]

### Classificacao Cross-Module

- [ ] Esta task toca APENAS 1 repositorio? -> Prosseguir normalmente
- [ ] Esta task toca 2+ repositorios? -> PARTIR em sub-tasks (ver regras no CLAUDE.md)
- [ ] Esta task depende de algo que NAO EXISTE noutro modulo? -> CRIAR spec da dependencia primeiro

---

## Fora de Scope

**NAO sera implementado nesta task**:
- [Funcionalidade A - sera feita em task separada]
- [Funcionalidade B - decisao futura]

---

## Quality Gate: SPECIFY

### Verificacao de Scope
- [ ] Task faz UMA coisa bem definida
- [ ] Requisitos sao independentes de implementacao
- [ ] Nao ha generalizacao prematura ("suportar futuramente...")
- [ ] Nao mistura API + logica + dados na mesma task

### Servicos/Packages Existentes Relacionados
| Servico/Package | Relacao | Reutilizar? |
|-----------------|---------|-------------|
| [pesquisar...] | [similar/base/nenhuma] | [sim/nao] |

### Decisao de Complexidade
- Novo servico/package necessario? [Sim/Nao] - Justificacao: [...]
- Nova interface necessaria? [Sim/Nao] - Quantas impl previstas: [...]
- Risco de over-engineering? [Baixo/Medio/Alto] - Mitigacao: [...]

### Entry-Points e Storage (AP-09, AP-10)
- [ ] Seccao "Entry-Points" preenchida com TODOS os caminhos do user ate a feature, OU marcada N/A com justificacao (feature puramente interna)
- [ ] Smoke navegavel descrito em 1 frase (vai virar smoke obrigatorio em /sdd-check)
- [ ] Para cada campo persistido: forma canonica armazenada declarada na seccao "Storage Semantics"
- [ ] Para cada path de escrita: conversao para forma canonica documentada
- [ ] Se a feature persiste state: 4 edge cases obrigatorios (EC-RT1..RT4) presentes ou justificadamente removidos
- [ ] Se altera semantics canonical de campo ja existente: plano de migracao documentado

---

## Questoes em Aberto

- [ ] [Questao 1 que precisa de clarificacao]
- [ ] [Questao 2 pendente de decisao]

---

## Historico

| Data | Autor | Alteracao |
|------|-------|-----------|
| [DATA] | [NOME] | Criacao inicial |
