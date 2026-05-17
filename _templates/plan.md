# Plan: [TASK-NAME]

> **Estado**: DRAFT | PLANNED
> **Spec**: [Link para spec.md]
> **Classificacao**: [PLATFORM] | [APP] | [ADMIN] | [SDK]
> **Repositorio**: densare-platform | densare-apps | densare-admin | densare-sdk-dotnet
> **Criado**: [DATA]

---

## Resumo do Plano

[Descricao breve da abordagem de implementacao em 2-3 frases]

---

## Analise da Spec

### Requisitos Mapeados
| Requisito | Complexidade | Prioridade |
|-----------|--------------|------------|
| RF-01 | Baixa/Media/Alta | P1/P2/P3 |
| RF-02 | ... | ... |

### Riscos Identificados
| Risco | Probabilidade | Impacto | Mitigacao |
|-------|---------------|---------|-----------|
| [Risco 1] | Alta/Media/Baixa | Alto/Medio/Baixo | [Estrategia] |

---

## Arquitectura

### Componentes Envolvidos
```
┌─────────────────┐     ┌─────────────────┐
│   [Componente]  │────>│   [Componente]  │
└─────────────────┘     └─────────────────┘
```

### Decisoes de Design
1. **[Decisao 1]**: [Justificacao]
2. **[Decisao 2]**: [Justificacao]

---

## Analise do Codigo Existente

> **OBRIGATORIO**: Antes de planear ficheiros novos, investigar o que ja existe.
> Ver Regra Zero em `sdd/QUALITY_GATES.md`.

### Pesquisa Realizada

| O que procurei | Onde procurei | O que encontrei |
|----------------|---------------|-----------------|
| [funcionalidade/conceito] | [pastas/ficheiros] | [resultado: existe/nao existe/similar] |

### Codigo Existente que Pode Ser Reutilizado/Modificado

| Ficheiro Existente | O que ja faz | O que precisa de mudar | Decisao |
|--------------------|-------------|------------------------|---------|
| `[path]` | [funcionalidade actual] | [alteracao necessaria] | Modificar / Estender / Nao serve |

---

## Ficheiros a Alterar

> **Prioridade**: Modificar existente > Estender existente > Criar novo.
> Se ha mais ficheiros novos do que modificados -> rever se nao ha codigo existente aproveitavel.

### Ficheiros Existentes a Modificar (PRIMEIRO)

| Ficheiro | O que Existe | O que Vai Mudar | Camada |
|----------|-------------|-----------------|--------|
| `[path]` | [funcionalidade actual] | [alteracao] | [Handler/Service/Repository] |

### Ficheiros Novos (so se justificado)

| Ficheiro | Camada | Proposito | Porque nao modificar existente? |
|----------|--------|-----------|--------------------------------|
| `[path]` | [Handler/Service/Repository] | [Descricao] | [Justificacao: nao existe nada similar / dominio diferente / etc.] |

---

## Wiring de Entry-Points (AP-09)

> Para CADA entry-point declarado na seccao "Entry-Points" da spec, indicar o ficheiro de wiring tocado neste plano. Se a spec declara entry-points mas este plano nao toca nenhum ficheiro de wiring -> plano incompleto.

| Entry-Point (da spec) | Ficheiro de wiring | Linha aproximada | Tipo de mudanca |
|-----------------------|--------------------|-----------------:|-----------------|
| [ex: Menu "Relatorios -> Caderno"] | `DenThermUIProvider.Menus.cs` | ~194 | Adicionar TabDefinition |
| [ex: Nav node "PTL"] | `Navigation.cs` | ~80 | Novo nav node debaixo de "Envolvente" |
| [ex: Route POST /v1/links] | `cmd/2snip/main.go` | router setup | `r.Post("/v1/links", h.Create)` |

---

## Helpers de Storage Canonical (AP-10)

> Para CADA campo persistido com 2+ write paths declarado na seccao "Storage Semantics" da spec, indicar o helper centralizado. Se a spec declara 2+ write paths mas este plano nao planeia helper centralizado -> plano incompleto.

| Campo | Helper centralizado | Localizacao | Conversao aplicada |
|-------|---------------------|-------------|---------------------|
| [ex: U-value opaco] | `OpaqueElementUValueService.ApplyAggravation` | `Application/Services/` | base * 1.35 quando AggravateU=true |
| [ex: Subscription period] | `extendSubscriptionPeriod` | `internal/payment/service_webhook.go` | ledger-based credit/carry |

Helpers a reutilizar (ja existem):
- [helper] em [path] — usado para [campo]

---

## Passos de Implementacao

### Passo 1: [Nome do Passo]
**Objetivo**: [O que este passo alcanca]

**Acoes**:
1. [ ] [Acao especifica]
2. [ ] [Acao especifica]

**Ficheiros**:
- `[path/to/file]`

**Validacao**:
- [ ] [Como verificar que este passo esta completo]

---

### Passo 2: [Nome do Passo]
**Objetivo**: [O que este passo alcanca]

**Acoes**:
1. [ ] [Acao especifica]
2. [ ] [Acao especifica]

**Ficheiros**:
- `[path/to/file]`

**Validacao**:
- [ ] [Como verificar que este passo esta completo]

---

### Passo Final (OBRIGATORIO): Wire Entry-Points + Smoke Navegavel (AP-09)

**Objetivo**: Garantir que a feature e acessivel ao utilizador/cliente externo a partir do estado inicial. Sem este passo, o resto do plano produz codigo invisivel — AP-09.

**Acoes**:
1. [ ] Wire de TODOS os entry-points listados na seccao "Wiring de Entry-Points" acima
2. [ ] Run navigable smoke do estado inicial ate a feature, conforme spec
3. [ ] Confirmar que o smoke passa SEM ler codigo (so seguindo a UI/cliente)

**Validacao**:
- [ ] Smoke navegavel da spec executado e a passar
- [ ] Cada entry-point produz UX/comportamento esperado

> Se a feature e puramente interna (spec marcou Entry-Points: N/A com justificacao), este passo pode ser omitido — documentar a omissao explicitamente.

---

## Testes

### Testes Unitarios
| Teste | Descricao | Ficheiro |
|-------|-----------|----------|
| `Test_[Nome]` | [O que valida] | `[path]_test.go` |

### Testes de Integracao
| Cenario | Descricao |
|---------|-----------|
| [Cenario] | [O que valida] |

### Testes de Seguranca
| Teste | Descricao |
|-------|-----------|
| JWT invalido | Retorna 401 |
| Rate limit excedido | Retorna 429 |
| Input malicioso | Retorna 400 |

### Testes Round-Trip / Storage (AP-10)

> Obrigatorios para features que persistem state. Eliminam por construcao a classe DT-572..578.

| Teste | Descricao |
|-------|-----------|
| `Test_NoOpEdit_PreservesAllFields` | Abrir entidade + Save sem alterar nada -> 100% dos campos preservados |
| `Test_RoundTrip_SaveReopenSaveIsIdempotent` | Save -> reload -> Save -> diff == empty |
| `Test_ImportVsNew_ProduceSameStored` | Importar legacy + criar novo do zero -> stored estructuralmente igual |
| `Test_MassUpdate_PreservesUntouchedFieldSemantics` | Mass-update em campo X sobre item importado -> campo Y (nao tocado) preserva forma canonica do path import |
| `Test_CanonicalForm_Per_WritePath` | Para CADA write path declarado em Storage Semantics, assert que stored value respeita a forma canonica declarada |

### Smoke Manual Navegavel (AP-09)

> Obrigatorio para features observaveis pelo utilizador / cliente externo. Replicar o "smoke navegavel" da seccao Entry-Points da spec.

| # | Smoke | Pre-requisito | Passos (1 por linha) | Resultado esperado |
|---|-------|---------------|----------------------|--------------------|
| Smoke-1 | [Acesso via menu] | App fresca | 1. Abrir app. 2. Click menu X. 3. Click sub-item Y. | Painel Z renderiza com [dados] |

---

## Dependencias de Ordem

```
Passo 1 ──> Passo 2 ──> Passo 3
                   └──> Passo 4 (paralelo)
```

---

## Estimativa

| Passo | Pontos |
|-------|--------|
| Passo 1 | [X] |
| Passo 2 | [X] |
| **Total** | **[X]** |

---

## Quality Gate: PLAN

### Verificacao de Camadas
| Ficheiro Novo | Camada Proposta | Correta? | Notas |
|---------------|-----------------|----------|-------|
| `[path]` | Handler/Service/Repository | [Sim/Nao] | |

### Verificacao Search Before Create
- [ ] Pesquisa de codigo existente foi feita? (seccao "Analise do Codigo Existente" preenchida)
- [ ] Cada ficheiro novo tem justificacao de porque nao se pode modificar existente?
- [ ] Ha mais ficheiros modificados do que novos? (se nao, rever)

### Packages Existentes a Reutilizar
| Necessidade | Existe em pkg/? | Package | Acao |
|-------------|-----------------|---------|------|
| Middleware JWT | Verificar... | | Reutilizar/Criar |
| Error types | Verificar... | | Reutilizar/Criar |
| Config loading | Verificar... | | Reutilizar/Criar |

### Padrao Escolhido
| Aspeto | Padrao Existente | Padrao Escolhido | Justificacao |
|--------|------------------|------------------|--------------|
| Error handling | [existente] | [escolhido] | [porque] |
| Validacao | [existente] | [escolhido] | [porque] |
| Logging | [existente] | [escolhido] | [porque] |

### Verificacao de Interfaces
| Interface Nova | Implementacoes Previstas | Justificada? |
|----------------|--------------------------|--------------|
| [nome] | [lista] | [Sim/Nao] |

### Verificacao de Seguranca
- [ ] Endpoints publicos tem rate limiting planeado?
- [ ] JWT validation planeada para endpoints autenticados?
- [ ] Input validation planeada nos handlers?
- [ ] SQL parametrizado planeado nas queries?
- [ ] Segredos via env vars?

### Verificacao de Entry-Points (AP-09)
- [ ] Para CADA entry-point da spec, ha ficheiro de wiring listado na seccao "Wiring de Entry-Points"
- [ ] Existe "Passo Final: Wire Entry-Points + Smoke Navegavel" na ordem de implementacao
- [ ] Smoke navegavel da spec esta enumerado nos testes manuais
- [ ] Se feature e puramente interna (Entry-Points: N/A na spec): omissao do passo final esta justificada

### Verificacao de Storage Semantics (AP-10)
- [ ] Para CADA campo persistido com 2+ write paths (da seccao "Storage Semantics" da spec), helper centralizado planeado
- [ ] Helpers reutilizam codigo existente quando possivel (AP-04)
- [ ] Testes round-trip (no-op edit, save->reload, import-vs-new, mass-update) presentes na seccao Testes
- [ ] Se feature nao persiste state (spec marcou Storage: N/A): omissao justificada

---

## Dependencias Cross-Module

> Se esta task depende de ou afecta outros modulos/repositorios.

### Pre-condicoes (o que precisa de existir ANTES de implementar)

| Dependencia | Modulo | Estado Actual | Spec | Bloqueante? |
|-------------|--------|---------------|------|-------------|
| [endpoint/funcao/modelo] | [PLATFORM]/[APP]/[ADMIN]/[SDK] | [estado] | [path] | [Sim/Nao] |

### Impacto noutros modulos (o que esta task desbloqueia)

| Modulo Afectado | O que fica disponivel | Spec dependente |
|-----------------|----------------------|-----------------|
| [modulo] | [endpoint/funcao/modelo] | [path spec] |

### Ordem de Implementacao Cross-Module

```
[Se aplicavel: diagrama de sequencia entre modulos]
1. [PLATFORM] spec-X -> IMPLEMENTAR PRIMEIRO
2. [ADMIN] spec-Y -> IMPLEMENTAR DEPOIS (depende de 1)
```

---

## Checklist Pre-Implementacao

- [ ] Spec aprovada (SPECIFIED)
- [ ] Dependencias tecnicas disponiveis
- [ ] Dependencias cross-module satisfeitas (PLANNED ou IMPLEMENTED)
- [ ] Tasks dependentes completas
- [ ] Quality Gates verificados
- [ ] Estimativa aprovada

---

## Notas de Implementacao

[Notas adicionais, gotchas, referencias uteis]

---

## Historico

| Data | Autor | Alteracao |
|------|-------|-----------|
| [DATA] | [NOME] | Criacao inicial |
