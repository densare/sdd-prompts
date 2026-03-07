# Quality Gates - SDD Densare AEC

Este documento define verificacoes de qualidade obrigatorias em cada fase do SDD, adaptadas para C# / .NET e aplicacoes desktop.

> **Pre-requisito**: Ler `sdd/ANTI_PATTERNS.md` antes de qualquer fase tecnica (/sdd-plan, /implement, /check). Contem exemplos REAIS de problemas a evitar.

---

## Regra Zero: SEARCH BEFORE CREATE

> **Antes de planear ou criar qualquer ficheiro, classe, servico ou namespace: investigar o codigo existente.**
> A preferencia e SEMPRE modificar codigo existente em vez de criar novo.

### Como investigar (obrigatorio durante /sdd-plan)

```
Para CADA requisito da spec, fazer por esta ordem:

1. PROCURAR no codigo-fonte (Grep/Glob):
   - Classes, services, ViewModels que facam algo similar
   - Namespaces que tratem do mesmo dominio
   - Interfaces que ja existam para o mesmo tipo de operacao

2. ANALISAR o que encontrou:
   - Este codigo pode ser ESTENDIDO para cobrir o novo requisito?
   - Precisa de pequenas alteracoes (novo metodo, novo parametro)?
   - Ou e realmente algo diferente que nao se encaixa?

3. DECIDIR (por esta prioridade):
   a) MODIFICAR codigo existente — se o que existe pode ser ajustado (preferencia)
   b) ESTENDER codigo existente — adicionar metodo a classe/namespace existente
   c) GENERALIZAR — se 2+ sitios fazem algo similar, extrair para partilhado
   d) CRIAR novo — APENAS se nada do existente serve (justificar no plan.md)

4. DOCUMENTAR no plan.md:
   - O que foi encontrado (ficheiros, classes, namespaces)
   - Decisao tomada (modificar / estender / generalizar / criar)
   - Justificacao se decidir criar novo
```

### Onde procurar

```
C# / .NET:
  - <Modulo>.Domain/        — Entidades, Value Objects existentes
  - <Modulo>.Application/   — Services, Interfaces existentes
  - <Modulo>.Infrastructure/ — Repositorios, Data access existentes
  - <Modulo>.Presentation/  — ViewModels, Views existentes
  - Core/                   — Packages partilhados
```

### Red Flags

```
Se o plan.md tem MAIS ficheiros novos do que ficheiros modificados -> PARAR e rever.
Num projecto existente, a maioria das features deve modificar/estender codigo, nao criar do zero.
```

Esta regra aplica-se em TODAS as fases: SPECIFY, PLAN, IMPLEMENT, CHECK.

---

## Regra de Proporcionalidade

> **A complexidade da solucao DEVE ser proporcional a complexidade do problema.**

| Tipo de Dados | Seguranca Adequada | Seguranca Excessiva (AP-02) |
|---------------|-------------------|----------------------------|
| Licencas, tokens, credenciais | DPAPI, validacao | - |
| Settings, estado de UI | Permissoes basicas | Encriptacao, audit |
| Dados de projecto (calculos) | Validacao de dominio | Encriptacao |

Antes de adicionar seguranca, responder: "Qual e a ameaca concreta? Quem atacaria, como, porque?"

---

## Limites Concretos de Codigo

| Metrica | C# | Se exceder |
|---------|----|-----------|
| LOC por ficheiro | < 500 🟢 / 500-600 🟡 / > 600 🔴 | 🟡 congelar, 🔴 split |
| LOC por metodo | < 45 🟢 / 45-55 🟡 / > 55 🔴 | 🟡 congelar, 🔴 split |
| Metodos publicos por classe | 15 | PARAR, split |
| Parametros por metodo | 5 | Considerar record/options |
| Interfaces com 1 impl | 0 | Justificar ou remover |

**Excepcoes aceites para interfaces 1:1**: boundary de teste (repository -> mock) ou boundary de sistema (API externa -> trocar provider). Documentar justificacao.

---

## Problemas Conhecidos a Prevenir

> Ver `sdd/ANTI_PATTERNS.md` para exemplos detalhados com codigo real.

| ID | Problema | Resumo | Gate |
|----|----------|--------|------|
| AP-01 | Interfaces 1:1 | Interface para cada classe sem polimorfismo | PLAN, IMPLEMENT |
| AP-02 | Seguranca desproporcional | AES-256 para window positions | PLAN, IMPLEMENT |
| AP-03 | Security theater | [Authorize] sem middleware que enforce | IMPLEMENT, CHECK |
| AP-04 | Duplicacao sistematica | Mesma logica copiada em 4 modulos | SPECIFY, PLAN |
| AP-05 | God objects | Ficheiros com 700+ LOC | IMPLEMENT, CHECK |
| AP-06 | Codigo na camada errada | SQL em ViewModels, UI em Domain | PLAN, IMPLEMENT |
| AP-07 | Dead code | Stubs vazios, features fantasma | IMPLEMENT, CHECK |
| AP-08 | Multiplos padroes | 5 formas de fazer a mesma coisa | PLAN, IMPLEMENT |

---

## Gate 1: SPECIFY - Prevencao de Over-engineering

### Passo 0: Search Before Create + Verificacao Cross-Module

```
OBRIGATORIO antes de especificar:
1. Procurar classes/namespaces existentes que facam o mesmo (AP-04)
2. Listar explicitamente o que ja existe e pode ser reutilizado
3. Se existe algo similar -> justificar porque nao serve
4. Verificar se o request pertence ao modulo onde esta guardado
5. Verificar se o request precisa de funcionalidade de outro modulo
6. Se toca 2+ repositorios -> PARTIR em sub-tasks
```

### Checklist Obrigatoria

**Scope**
- [ ] A task faz UMA coisa bem definida?
- [ ] Os requisitos sao independentes de implementacao?
- [ ] Ha requisitos que deveriam ser tasks separadas?

**Complexidade e Proporcionalidade**
- [ ] Justifica criar novo namespace/modulo? (AP-04: search first)
- [ ] Existe algo reutilizavel que faca o mesmo?
- [ ] Esta a generalizar prematuramente para casos futuros? (AP-01)
- [ ] A seguranca pedida e proporcional ao tipo de dados? (AP-02)

**Classificacao e Cross-Module**
- [ ] [CORE], [THERMAL], [SIM] ou [APP]?
- [ ] Repositorio destino identificado?
- [ ] Request esta no modulo correcto? (se nao, mover ou referenciar)
- [ ] Precisa de funcionalidade de outro modulo? (se sim, documentar dependencia)
- [ ] Toca 2+ repositorios? (se sim, PARTIR em sub-tasks)

---

## Gate 2: PLAN - Prevencao de Violacoes Arquitecturais

### Passo 0: Ler ANTI_PATTERNS.md + Verificar Dependencias

```
OBRIGATORIO antes de planear:
1. Ler sdd/ANTI_PATTERNS.md (especialmente AP-01, AP-04, AP-06, AP-08)
2. Para cada ficheiro novo no plano, verificar se ja existe algo similar
3. Para cada interface nova, documentar quantas implementacoes tera
4. Definir padroes a usar (e verificar se o projecto ja tem padrao)
5. Verificar dependencias cross-module da spec
6. Criar grafo de dependencias entre passos (identificar paralelizacao)
7. Documentar riscos de implementacao e plano de rollback
```

### Checklist Obrigatoria

**Camadas C# (Clean Architecture)**
- [ ] Cada ficheiro novo esta na camada correta?
- [ ] ViewModels so fazem: binding, commands, chamar services?
- [ ] Services contem TODA a logica de aplicacao?
- [ ] Repositories sao a UNICA camada com EF Core/SQL?
- [ ] Domain nao depende de nada externo?

```
Mapeamento de Camadas C#:
Presentation  -> ViewModels, Views, Controls (UI/Avalonia)
Application   -> Services, DTOs, Validators (logica de orquestracao)
Domain        -> Entities, ValueObjects, Enums (regras de dominio)
Infrastructure -> Repositories, EF Core, External APIs (I/O)
```

**Reutilizacao (AP-04: Search Before Create)**
- [ ] Procurou em Core/ por codigo que faca algo similar?
- [ ] O novo codigo pode ir para Core (reutilizavel) ou especifico do modulo?
- [ ] Estamos a criar duplicacao de services, DTOs, etc?
- [ ] Se o codigo sera usado por 2+ modulos -> esta em Core?

**Interfaces (AP-01: Interfaces 1:1)**
- [ ] Cada interface nova tem 2+ implementacoes previstas?
- [ ] Se so 1 implementacao -> usar classe directa (sem interface)
- [ ] Excepcao documentada? (boundary de teste ou de sistema)

**Padroes (AP-08: Multiplos Padroes)**
- [ ] Verificou que padrao o projecto ja usa para este problema?
- [ ] Segue o padrao existente?
- [ ] Se padrao novo, justificou porque os existentes nao servem?
- [ ] UM padrao por problema — nunca dois em paralelo

**Limites de Complexidade**
- [ ] Ficheiros na zona verde (< 500 LOC)? (AP-05)
- [ ] Métodos na zona verde (< 45 LOC)? (AP-05)
- [ ] Zero stubs ou "implementar depois"? (AP-07)

---

## Gate 3: IMPLEMENT - Prevencao de Code Smells

### Passo 0: Ler ANTI_PATTERNS.md + RED/GREEN/VERIFY

```
OBRIGATORIO antes de implementar:
1. Ler sdd/ANTI_PATTERNS.md
2. Ter presente as 9 perguntas de auto-verificacao (fim do ficheiro)
3. SEARCH BEFORE CREATE: procurar codigo existente antes de criar novo
4. Para cada RF-XX: RED (teste que falha) -> GREEN (codigo minimo) -> VERIFY (testes passam)
5. Se plan.md tem grafo de dependencias: respeitar ordem e paralelizacao
```

### Checklist C#

**Estrutura (AP-05: God Objects)**
- [ ] Classe tem UMA responsabilidade clara?
- [ ] Métodos > 55 LOC (zona vermelha)? -> PARAR, split obrigatório
- [ ] Ficheiro > 600 LOC (zona vermelha)? -> PARAR, split obrigatório

**Interfaces (AP-01: Interfaces 1:1)**
- [ ] Interface nova tem 2+ implementacoes concretas?
- [ ] Se so 1 implementacao -> usar classe directa (BLOQUEAR)
- [ ] Excepcao? Documentar: "boundary de teste" ou "boundary de sistema"

**Async/Await**
- [ ] Metodos I/O sao async?
- [ ] CancellationToken em metodos async publicos?
- [ ] Await em todos os Task (nao fire-and-forget sem razao)?

**Patterns MVVM**
- [ ] ViewModels herdam de ObservableObject?
- [ ] Commands usam RelayCommand/AsyncRelayCommand?
- [ ] Propriedades reactivas usam [ObservableProperty]?

**Seguranca**
- [ ] Validacao de input nos ViewModels?
- [ ] Dados sensiveis protegidos (DPAPI se necessario)?
- [ ] Excepcoes nao expoe detalhes ao utilizador?

### Red Flags - PARAR IMEDIATAMENTE

| Situacao | Anti-Pattern | Acao |
|----------|-------------|------|
| ViewModel com SQL | AP-06 | Mover para Repository |
| Domain com dependencias externas | AP-06 | Remover dependencia |
| Interface com 1 implementacao | AP-01 | Usar classe directa |
| Ficheiro > 600 LOC | AP-05 | Split imediato |
| Encriptacao de dados nao-sensiveis | AP-02 | Remover, simplificar |
| Stub/NotImplemented | AP-07 | Implementar ou nao criar |
| Padrao novo para problema ja resolvido | AP-08 | Usar padrao existente |

---

## Gate 4: CHECK - Verificacao de Qualidade

### Passo 0: Revisao Independente + Ler ANTI_PATTERNS.md

```
OBRIGATORIO antes de verificar:
0. REVISAO INDEPENDENTE: /sdd-check NUNCA na mesma sessao que /sdd-implement
   - Preferido: sessao Claude nova (sem memoria da implementacao)
   - Alternativa: repo alternativo (A/B) apos git pull
   - Minimo: se mesma sessao inevitavel, declarar no relatorio
1. Ler sdd/ANTI_PATTERNS.md
2. Usar as 9 perguntas de auto-verificacao como checklist
3. Contar interfaces vs implementacoes (AP-01)
4. Verificar LOC dos ficheiros criados (AP-05)
5. Relatorio em formato TABULAR (PASS/FAIL por criterio) — ver prompt check.md
```

### Checklist de Review

**Arquitectura (AP-06)**
- [ ] Codigo esta na camada correta? (Presentation/Application/Domain/Infrastructure)
- [ ] Dependencias respeitam direccao? (Presentation -> Application -> Domain)
- [ ] Codigo partilhado esta em Core?
- [ ] Nenhum ViewModel contem logica de dominio?
- [ ] Nenhum Domain contem dependencias de infraestrutura?

**Over-engineering (AP-01, AP-02)**
- [ ] CONTAR: Quantas interfaces? Quantas com 1 implementacao?
- [ ] Cada interface com 1 impl tem justificacao documentada?
- [ ] Nao ha abstraccoes "para o futuro"?
- [ ] Complexidade proporcional ao problema?
- [ ] Seguranca proporcional ao tipo de dados?

**Duplicacao (AP-04)**
- [ ] Procurar classes/metodos duplicados entre namespaces
- [ ] Codigo partilhado esta em Core?
- [ ] Nenhum copy-paste entre modulos?

**Dead Code (AP-07)**
- [ ] Zero stubs ou NotImplementedException?
- [ ] Tudo o que foi criado e usado?
- [ ] Nenhum event handler para eventos que nao existem?

**Limites de Complexidade (AP-05)**
- [ ] Ficheiros: 🟢 < 500 | 🟡 500-600 (congelado) | 🔴 > 600 (split)?
- [ ] Métodos: 🟢 < 45 | 🟡 45-55 (congelado) | 🔴 > 55 (split)?
- [ ] Nenhuma classe > 15 metodos publicos?

**Testes**
- [ ] Testes cobrem os requisitos da spec?
- [ ] Testes verificam OUTPUT, nao estado interno?
- [ ] Cenarios testados sao REALISTAS?

**Retrospetiva**
```
1. "Se fosse implementar de novo, faria igual?"
2. "Que alternativa mais simples existia?"
3. "Este codigo vai ser facil de modificar daqui a 6 meses?"
4. "Algum anti-pattern do ANTI_PATTERNS.md foi introduzido?"
```

---

## Gate 5: TESTES - Filosofia de Testing

### Principio Fundamental

> **Queremos o minimo suficiente para validar funcionalidade relevante, nao metricas de cobertura.**

### O que Testar

| Testar | Nao Testar |
|--------|------------|
| Outputs/resultados de services | Estado interno de classes |
| Cenarios de negocio reais | Edge cases irreais |
| Calculos termicos SCE | Getters/setters triviais |
| Validacoes de dominio | Constantes |
| Integracoes (EF Core) | Mocks que testam mocks |

### Testes C#

- xUnit para unit tests
- FluentAssertions para assertions legiveis
- Moq para mocks
- EF Core InMemory para testes de repositorio
- Testar services isolados (logica de aplicacao)
- Testar calculos de dominio (formulas SCE)
- BDD para cenarios complexos

### Red Flags em Testes

```
SE o teste:
- Testa estado interno -> TESTAR output
- Duplica outro teste -> ELIMINAR
- Testa constante trivial -> ELIMINAR
- Precisa de hack no codigo de producao -> REDESENHAR
```

---

## Gate 6: ARCHIVE - Aceitacao pelo Utilizador

### Passo 0: Verificar que task esta em concluidos/

```
OBRIGATORIO antes de gerar plano de aceitacao:
1. Verificar que a task esta em concluidos/ (ja passou por /sdd-close)
2. Verificar que close-report.md existe e tem veredicto APROVADO
3. Ler request.md e spec.md para gerar cenarios de teste
```

### Checklist Obrigatoria

**Plano de Teste**
- [ ] Sumario em linguagem simples (nao tecnica)?
- [ ] Localizacao na aplicacao descrita (menu, painel, atalho)?
- [ ] Expectativas visuais descritas (o que o user deve ver)?
- [ ] Exemplo completo end-to-end com dados realistas?
- [ ] Cenario de teste para CADA RF-XX da spec?
- [ ] Passos atomicos (uma accao por passo)?
- [ ] Resultados esperados claros e verificaveis?
- [ ] Casos negativos/edge cases incluidos?

**Veredictos**
```
ACEITE:
- Todos os cenarios OK
- Mover para arquivados/

ACEITE COM RESERVAS:
- Maioria dos cenarios OK, defeitos menores
- Criar issues Linear para defeitos
- Mover para arquivados/

REJEITADO:
- Defeitos criticos identificados
- Criar issues Linear para defeitos
- NAO mover — fica em concluidos/ ate correccao
- Apos fix: re-executar /sdd-archive
```

---

## Gate 7: CLOSE - Verificacao de Conclusao

### Passo 0: Verificar Linear

```
OBRIGATORIO antes de arquivar:
1. Identificar TODOS os issues Linear referenciados no plan.md
2. Verificar que TODOS estao Done (ou Cancelled com justificacao)
3. Se algum nao esta Done -> PARAR (nao arquivar parcialmente)
```

### Checklist Obrigatoria

**Issues Linear**
- [ ] Todos os issues referenciados no plan.md identificados?
- [ ] Todos Done ou Cancelled com justificacao?
- [ ] PRs associados estao merged?

**Mini-Auditoria (verificacao rapida, nao exaustiva)**
- [ ] Para cada RF-XX da spec: existe evidencia no codigo? (Grep rapido)
- [ ] Testes mencionados no plan.md existem?
- [ ] Nenhum requisito critico ficou por implementar?

**Documentacao**
- [ ] close-report.md gerado com veredicto?
- [ ] STATUS.md actualizado (task removida ou marcada como arquivada)?

### Criterios de Aprovacao

```
APROVADO:
- Todos os issues Done/Cancelled
- Requisitos criticos cobertos (mini-auditoria OK)
- close-report.md gerado

REPROVADO (nao mover):
- Issues em aberto
- Requisitos criticos sem evidencia
- PRs nao merged
```

---

## Metricas de Sucesso

### Estrutura de Codigo

| Metrica | Target | Red Flag | Anti-Pattern |
|---------|--------|----------|-------------|
| LOC por ficheiro | < 500 🟢 | > 600 🔴 | AP-05 |
| LOC por metodo | < 45 🟢 | > 55 🔴 | AP-05 |
| Metodos por classe | < 15 publicos | > 15 | AP-05 |
| Parametros por metodo | < 5 | > 5 | - |
| Camada correta | 100% | SQL em ViewModel | AP-06 |
| Interfaces com 1 impl | 0 (sem excepcao documentada) | > 0 | AP-01 |
| Padroes por problema | 1 | > 1 | AP-08 |
| Dead code / stubs | 0 | > 0 | AP-07 |

---

*Densare AEC SDD - Fevereiro 2026*
