# Spec: [TASK-NAME]

> **Estado**: DRAFT | SPECIFIED
> **Classificacao**: [PLATFORM] | [APP] | [ADMIN] | [SDK]
> **Repositorio**: densare-platform | densare-apps | densare-admin | densare-sdk-dotnet
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

### RF-01: [Nome do Requisito]
**Descricao**: [O que o sistema deve fazer]

**Criterios de Aceitacao**:
- [ ] Dado [contexto], quando [acao], entao [resultado esperado]
- [ ] Dado [contexto], quando [acao], entao [resultado esperado]

### RF-02: [Nome do Requisito]
**Descricao**: [O que o sistema deve fazer]

**Criterios de Aceitacao**:
- [ ] ...

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

---

## Questoes em Aberto

- [ ] [Questao 1 que precisa de clarificacao]
- [ ] [Questao 2 pendente de decisao]

---

## Historico

| Data | Autor | Alteracao |
|------|-------|-----------|
| [DATA] | [NOME] | Criacao inicial |
