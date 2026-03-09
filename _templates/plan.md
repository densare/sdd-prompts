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
