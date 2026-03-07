# Quality Gates - SDD Densare Cloud

Este documento define verificacoes de qualidade obrigatorias em cada fase do SDD, adaptadas para as stacks Go (+ Templ + HTMX) e C#/.NET 8.

> **Pre-requisito**: Ler `sdd/ANTI_PATTERNS.md` antes de qualquer fase tecnica. Contem exemplos REAIS de problemas a evitar.

---

## Regra Zero: SEARCH BEFORE CREATE

> **Antes de planear ou criar qualquer ficheiro, funcao, tipo ou package: investigar o codigo existente.**
> A preferencia e SEMPRE modificar codigo existente em vez de criar novo.

### Como investigar (obrigatorio durante /plan)

```
Para CADA requisito da spec, fazer por esta ordem:

1. PROCURAR no codigo-fonte (Grep/Glob):
   - Funcoes, tipos, handlers que facam algo similar
   - Packages/modulos que tratem do mesmo dominio
   - Endpoints que ja existam para o mesmo recurso

2. ANALISAR o que encontrou:
   - Este codigo pode ser ESTENDIDO para cobrir o novo requisito?
   - Precisa de pequenas alteracoes (novo campo, novo metodo, novo parametro)?
   - Ou e realmente algo diferente que nao se encaixa?

3. DECIDIR (por esta prioridade):
   a) MODIFICAR codigo existente — se o que existe pode ser ajustado (preferencia)
   b) ESTENDER codigo existente — adicionar funcao/metodo a package/modulo existente
   c) GENERALIZAR — se 2+ sitios fazem algo similar, extrair para partilhado
   d) CRIAR novo — APENAS se nada do existente serve (justificar no plan.md)

4. DOCUMENTAR no plan.md:
   - O que foi encontrado (ficheiros, funcoes, packages)
   - Decisao tomada (modificar / estender / generalizar / criar)
   - Justificacao se decidir criar novo
```

### Onde procurar

```
Go:
  - internal/<servico>/  — handlers, services, repositories do mesmo servico
  - internal/*/          — outros servicos que facam algo parecido
  - pkg/                 — packages partilhados
  - cmd/                 — pontos de entrada

Templ + HTMX:
  - templates/           — templates Templ existentes
  - templates/layouts/   — layouts base
  - templates/components/ — componentes reutilizaveis
  - static/              — assets estaticos (CSS, JS)

C#:
  - src/Densare.Client/  — classes existentes do SDK
  - Procurar por metodos, DTOs, handlers que tratem do mesmo recurso
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
| Passwords, tokens, dados pessoais | Encriptacao, auth, audit | - |
| Configuracao interna, estado de UI | Permissoes basicas | Encriptacao, rate limiting, audit |
| Preferencias visuais, temas | Nenhuma especial | Qualquer medida |

Antes de adicionar seguranca, responder: "Qual e a ameaca concreta? Quem atacaria, como, porque?"

---

## Limites Concretos de Codigo

| Metrica | Go / C# | Templ | Se exceder |
|---------|---------|-------|-----------|
| LOC por ficheiro | < 500 / 500-600 / > 600 | < 200 | congelar / split |
| LOC por funcao/metodo | < 45 / 45-55 / > 55 | < 45 | congelar / split |
| Funcoes exportadas por package | 15 | - | PARAR, split |
| Parametros por funcao | 5 | 5 | Considerar struct/options |
| Interfaces com 1 impl | 0 | - | Justificar ou remover |

**Excepcoes aceites para interfaces 1:1**: boundary de teste (repository -> mock) ou boundary de sistema (API externa -> trocar provider). Documentar justificacao.

---

## Problemas Conhecidos a Prevenir

> Ver `sdd/ANTI_PATTERNS.md` para exemplos detalhados com codigo real.

| ID | Problema | Resumo | Gate |
|----|----------|--------|------|
| AP-01 | Interfaces 1:1 | Interface para cada struct/class sem polimorfismo | PLAN, IMPLEMENT |
| AP-02 | Seguranca desproporcional | AES-256 para window positions | PLAN, IMPLEMENT |
| AP-03 | Security theater | [Authorize] sem middleware que enforce | IMPLEMENT, CHECK |
| AP-04 | Duplicacao sistematica | Mesma logica copiada em 4 modulos | SPECIFY, PLAN |
| AP-05 | God objects | Ficheiros que excedem limites LOC | IMPLEMENT, CHECK |
| AP-06 | Codigo na camada errada | SQL em handlers, UI em domain | PLAN, IMPLEMENT |
| AP-07 | Dead code | Stubs vazios, features fantasma | IMPLEMENT, CHECK |
| AP-08 | Multiplos padroes | 5 formas de fazer a mesma coisa | PLAN, IMPLEMENT |

> Para exemplos detalhados com codigo real de cada categoria, consultar `sdd/ANTI_PATTERNS.md`.

---

## Gate 1: SPECIFY - Prevencao de Over-engineering

### Passo 0: Search Before Create + Verificacao Cross-Module

```
OBRIGATORIO antes de especificar:
1. Procurar servicos/packages existentes que facam o mesmo (AP-04)
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
- [ ] Justifica criar novo servico/package? (AP-04: search first)
- [ ] Existe algo reutilizavel que faca o mesmo?
- [ ] Esta a generalizar prematuramente para casos futuros? (AP-01)
- [ ] A seguranca pedida e proporcional ao tipo de dados? (AP-02)

**Classificacao e Cross-Module**
- [ ] [PLATFORM], [APP], [ADMIN] ou [SDK]?
- [ ] Repositorio destino identificado?
- [ ] Request esta no modulo correcto? (se nao, mover ou referenciar)
- [ ] Precisa de funcionalidade de outro modulo? (se sim, documentar dependencia)
- [ ] Toca 2+ repositorios? (se sim, PARTIR em sub-tasks por repositorio)
- [ ] Dependencias cross-module preenchidas na spec?

**Perguntas Preventivas**
```
1. "Existe servico/package no projecto que ja faca isto?" (AP-04)
2. "Esta task e especifica para um servico ou deve ser partilhada?"
3. "A solucao e proporcional ao problema?" (AP-02)
4. "Estou a pedir seguranca para dados que nao sao sensiveis?" (AP-02)
```

**Red Flags na Spec**
- "Suportar futuramente..." -> Generalizacao prematura (AP-01)
- "Interface para..." sem multiplas implementacoes conhecidas (AP-01)
- "Encriptar/proteger/auditar..." sem threat model concreto (AP-02)
- Requisito que mistura API publica + logica + dados na mesma task
- Seguranca pedida sem identificar a ameaca concreta (AP-03)

---

## Gate 2: PLAN - Prevencao de Violacoes Arquitecturais

### Passo 0: Ler ANTI_PATTERNS.md + Verificar Dependencias

```
OBRIGATORIO antes de planear:
1. Ler sdd/ANTI_PATTERNS.md (especialmente AP-01, AP-04, AP-06, AP-08)
2. Para cada ficheiro novo no plano, verificar se ja existe algo similar
3. Para cada interface nova, documentar quantas implementacoes tera
4. Definir padroes a usar (e verificar se o projecto ja tem padrao para o problema)
5. Verificar dependencias cross-module da spec:
   - Se dependencia com estado NAO_EXISTE ou DRAFT -> BLOQUEAR
   - Se dependencia com estado SPECIFIED -> AVISAR (pode planear, nao pode implementar)
   - Preencher seccao "Dependencias Cross-Module" no plan.md
```

### Checklist Obrigatoria

**Camadas Go (densare-platform)**
- [ ] Cada ficheiro novo esta na camada correta?
- [ ] Handlers so fazem: parse input, chamar service, retornar response?
- [ ] Services contem TODA a logica de negocio?
- [ ] Repositories sao a UNICA camada com SQL/Redis?

```
Mapeamento de Camadas Go:
handler.go    -> HTTP (parse, validate input, call service, format response)
service.go    -> Logica de negocio (orquestrar, decidir, validar regras)
repository.go -> Dados (SQL, Redis, queries)
model.go      -> Structs, tipos, validacao de dominio
```

**Camadas Templ + HTMX (2snip, 2send, densare-admin)**
- [ ] Templates Templ so fazem rendering?
- [ ] Logica de negocio esta nos handlers/services Go?
- [ ] HTMX faz actualizacoes parciais (nao full page reload)?

```
Mapeamento de Camadas Templ + HTMX:
templates/       -> UI (Templ, rendering HTML)
handler.go       -> HTTP handlers (servem templates, processam forms)
service.go       -> Logica de negocio
static/          -> CSS (Tailwind), JS minimo (se necessario)
```

**Camadas C# (densare-sdk-dotnet)**
- [ ] Classe publica tem API limpa e documentada?
- [ ] Implementacao interna esta encapsulada?
- [ ] I/O usa async/await com CancellationToken?

```
Mapeamento de Camadas C#:
DensareClient.cs    -> API publica (fachada)
Auth/               -> Login, tokens, refresh (interno)
License/            -> Licencas, keep-alive (interno)
Http/               -> HttpClient, retry, backoff (interno)
Models/             -> DTOs publicos
```

**Reutilizacao (AP-04: Search Before Create)**
- [ ] Procurou em `pkg/` por package que faca algo similar?
- [ ] O novo codigo pode ir para `pkg/` (reutilizavel) ou `internal/` (especifico)?
- [ ] Estamos a criar duplicacao de middleware, error types, etc?
- [ ] Se o codigo sera usado por 2+ servicos -> esta em `pkg/`?

**Interfaces (AP-01: Interfaces 1:1)**
- [ ] Cada interface nova tem 2+ implementacoes previstas?
- [ ] Se so 1 implementacao -> usar struct directa (sem interface)
- [ ] Excepcao documentada? (boundary de teste ou de sistema)

**Padroes (AP-08: Multiplos Padroes)**
- [ ] Verificou que padrao o projecto ja usa para este problema?
- [ ] Segue o padrao existente?
- [ ] Se padrao novo, justificou porque os existentes nao servem?
- [ ] UM padrao por problema — nunca dois em paralelo

**Proporcionalidade de Seguranca (AP-02, AP-03)**
- [ ] Cada medida de seguranca tem threat model documentado?
- [ ] Nenhuma encriptacao/audit/rate-limit para dados nao-sensiveis?
- [ ] Cada atributo de seguranca tem enforcement verificavel?

**Limites de Complexidade**
- [ ] Ficheiros na zona verde (< 500 LOC)? (AP-05)
- [ ] Funcoes/metodos na zona verde (< 45 LOC)? (AP-05)
- [ ] Zero stubs ou "implementar depois"? (AP-07)

---

## Gate 3: IMPLEMENT - Prevencao de Code Smells

### Passo 0: Ler ANTI_PATTERNS.md

```
OBRIGATORIO antes de implementar:
1. Ler sdd/ANTI_PATTERNS.md
2. Ter presente as 9 perguntas de auto-verificacao (fim do ficheiro)
3. SEARCH BEFORE CREATE: procurar codigo existente antes de criar novo
```

### Checklist Go

**Estrutura (AP-05: God Objects)**
- [ ] Package tem UMA responsabilidade clara?
- [ ] Funcoes/metodos > 55 LOC (zona vermelha)? -> PARAR, split obrigatorio
- [ ] Ficheiro > 600 LOC (zona vermelha)? -> PARAR, split obrigatório

**Interfaces (AP-01: Interfaces 1:1)**
- [ ] Interface nova tem 2+ implementacoes concretas?
- [ ] Se so 1 implementacao -> usar struct directa (BLOQUEAR)
- [ ] Excepcao? Documentar: "boundary de teste" ou "boundary de sistema"

**Errors**
- [ ] Errors retornados com contexto (`fmt.Errorf("doing X: %w", err)`)?
- [ ] Usa error types de `pkg/errors/`?
- [ ] NAO usa panic em codigo de producao?

**Seguranca**
- [ ] SQL parametrizado (`$1, $2` nunca `%s`)?
- [ ] Input validado no handler antes de passar ao service?
- [ ] Segredos via env vars (`os.Getenv` ou config package)?
- [ ] Context propagado em toda a cadeia?

### Checklist Templ + HTMX

**Templates**
- [ ] Template tem UMA responsabilidade (componente ou pagina)?
- [ ] Logica de negocio esta no handler/service, nao no template?
- [ ] Dados passados como struct tipado (nao interface{})?

**Seguranca**
- [ ] Templ escapa HTML automaticamente (nao usar templ.Raw sem justificacao)?
- [ ] CSRF token em todos os formularios POST?
- [ ] Nao expoe dados sensiveis no HTML renderizado?

### Checklist C#

**API Publica**
- [ ] Metodos publicos async com CancellationToken?
- [ ] IDisposable implementado correctamente?
- [ ] XML docs nos metodos publicos?

**Robustez**
- [ ] Retry com Polly para chamadas HTTP?
- [ ] Timeout configuravel?
- [ ] Grace period para modo offline?

### Red Flags - PARAR IMEDIATAMENTE

| Situacao | Anti-Pattern | Acao |
|----------|-------------|------|
| Handler com logica de negocio | AP-06 | Mover para service |
| SQL no service | AP-06 | Mover para repository |
| `fmt.Sprintf` para queries SQL | Seguranca | Usar parametros `$1, $2` |
| Segredos hardcoded | Seguranca | Usar env vars |
| Package que duplica outro | AP-04 | Reutilizar existente |
| Interface com 1 implementacao | AP-01 | Usar struct directa |
| Ficheiro > 600 LOC | AP-05 | Split imediato |
| Encriptacao de dados nao-sensiveis | AP-02 | Remover, simplificar |
| Atributo de seguranca sem enforcement | AP-03 | Remover ou implementar handler |
| Stub/NotImplemented | AP-07 | Implementar ou nao criar |
| Padrao novo para problema ja resolvido | AP-08 | Usar padrao existente |
| Fetch directo em componente | AP-06 | Mover para lib/api/ |
| Sanitizer redundante com queries parametrizadas | AP-03 | Remover sanitizer |

---

## Gate 4: CHECK - Verificacao de Qualidade

### Passo 0: Ler ANTI_PATTERNS.md

```
OBRIGATORIO antes de verificar:
1. Ler sdd/ANTI_PATTERNS.md
2. Usar as 9 perguntas de auto-verificacao como checklist
3. Contar interfaces vs implementacoes (AP-01)
4. Verificar LOC dos ficheiros criados (AP-05)
```

### Checklist de Review

**Arquitectura (AP-06)**
- [ ] Codigo esta na camada correta? (handler/service/repository)
- [ ] Dependencias respeitam direccao? (handler -> service -> repository)
- [ ] Packages partilhados estao em `pkg/`?
- [ ] Nenhum handler contem logica de negocio?
- [ ] Nenhum service contem SQL ou HTTP?

**Over-engineering (AP-01, AP-02)**
- [ ] CONTAR: Quantas interfaces? Quantas com 1 implementacao?
- [ ] Cada interface com 1 impl tem justificacao documentada?
- [ ] Nao ha abstraccoes "para o futuro"?
- [ ] Complexidade proporcional ao problema?
- [ ] Seguranca proporcional ao tipo de dados?

**Duplicacao (AP-04)**
- [ ] Procurar funcoes/tipos duplicados entre packages
- [ ] Codigo partilhado esta em `pkg/` ou `lib/`?
- [ ] Nenhum copy-paste entre modulos?

**Seguranca Real (AP-03)**
- [ ] Todos os endpoints publicos validam JWT?
- [ ] Rate limiting configurado?
- [ ] Input validation nos handlers?
- [ ] SQL parametrizado?
- [ ] Segredos via env vars?
- [ ] CADA medida de seguranca tem enforcement verificavel?
- [ ] Nenhum sanitizer redundante com queries parametrizadas?

**Padroes (AP-08)**
- [ ] Um unico padrao por problema?
- [ ] Padrao consistente com resto do projecto?

**Dead Code (AP-07)**
- [ ] Zero stubs ou NotImplementedException?
- [ ] Tudo o que foi criado e usado?
- [ ] Nenhum event handler para eventos que nao existem?

**Limites de Complexidade (AP-05)**
- [ ] Ficheiros: 🟢 < 500 | 🟡 500-600 (congelado) | 🔴 > 600 (split)?
- [ ] Funcoes/metodos na zona verde (< 45 LOC)?
- [ ] Nenhum package > 15 funcoes exportadas?

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
| Outputs/resultados de services | Estado interno de structs |
| Cenarios de negocio reais | Edge cases irreais |
| Fluxos HTTP completos (handler -> response) | Getters/setters triviais |
| Seguranca (JWT invalido, rate limit) | Constantes |
| Integracoes (DB, Redis) | Mocks que testam mocks |

### Testes por Stack

**Go**:
- Table-driven tests (`_test.go` no mesmo package)
- `httptest` para testar handlers
- Test containers para PostgreSQL/Redis em integracao
- Benchmarks para endpoints criticos (keep-alive)

**Templ + HTMX**:
- `go test` para testar handlers que servem templates
- `httptest` para testar respostas HTML
- Testar fluxos criticos via HTTP (login, upload, formularios)

**C# / .NET 8**:
- xUnit para unit tests
- Moq para mocks
- Testar API publica do SDK (LoginAsync, KeepAlive, etc.)
- Testar modo offline e retry

### Red Flags em Testes

```
SE o teste:
- Testa estado interno -> TESTAR output
- Duplica outro teste -> ELIMINAR
- Testa constante trivial -> ELIMINAR
- Precisa de hack no codigo de producao -> REDESENHAR
```

---

## Accoes Automaticas por Fase

### Durante /specify
1. **LER** `sdd/ANTI_PATTERNS.md`
2. **SEARCH**: Listar servicos/packages similares existentes (AP-04)
3. Classificar funcionalidade ([PLATFORM], [APP], [ADMIN], [SDK])
4. **CROSS-MODULE**: Verificar se pertence ao modulo correcto
5. **CROSS-MODULE**: Verificar se precisa de funcionalidade de outro modulo
6. **CROSS-MODULE**: Se toca 2+ repositorios -> PARTIR em sub-tasks
7. Alertar se spec mistura responsabilidades
8. Verificar proporcionalidade de seguranca (AP-02)
9. Identificar repositorio destino

### Durante /plan
1. **LER** `sdd/ANTI_PATTERNS.md`
2. **CROSS-MODULE**: Verificar dependencias da spec — BLOQUEAR se dependencia NAO_EXISTE/DRAFT
3. **SEARCH**: Procurar codigo existente antes de planear ficheiros novos (AP-04)
4. Validar camadas contra arquitectura (handler/service/repository) (AP-06)
5. BLOQUEAR se criar package que duplica existente (AP-04)
6. EXIGIR justificacao para novas interfaces — quantas implementacoes? (AP-01)
7. Verificar proporcionalidade de seguranca (AP-02)
8. Definir padroes e verificar consistencia com projecto (AP-08)
9. Verificar limites LOC planeados (AP-05)
10. Preencher seccao "Dependencias Cross-Module" no plan.md

### Durante /implement
1. **LER** `sdd/ANTI_PATTERNS.md` — ter perguntas de auto-verificacao presentes
2. **CROSS-MODULE**: Verificar que pre-condicoes de outros modulos estao IMPLEMENTED
3. **SEARCH** antes de criar cada ficheiro/funcao novo (AP-04)
4. BLOQUEAR se interface com 1 implementacao sem justificacao (AP-01)
5. BLOQUEAR se SQL nao parametrizado
6. BLOQUEAR se segredos hardcoded
7. BLOQUEAR se ficheiro > 600 LOC ou template > 200 LOC (AP-05)
8. BLOQUEAR se seguranca sem enforcement (AP-03)
9. Alertar se handler tem logica de negocio (AP-06)
10. Verificar que testes existem

### Durante /check
1. **LER** `sdd/ANTI_PATTERNS.md`
2. **CONTAR** interfaces vs implementacoes (AP-01)
3. **VERIFICAR** LOC de todos os ficheiros criados (AP-05)
4. **PROCURAR** duplicacoes entre packages (AP-04)
5. **VERIFICAR** proporcionalidade de seguranca (AP-02)
6. **VERIFICAR** enforcement de cada medida de seguranca (AP-03)
7. Comparar com checklist de qualidade
8. Gerar relatorio com seccao "Anti-Patterns Detectados"
9. Retrospetiva

### Durante /close
1. **LER** spec.md e plan.md da task
2. **IDENTIFICAR** todos os issues Linear
3. **VERIFICAR** estado de cada issue no Linear
4. **PARAR** se algum nao esta Done
5. **MINI-AUDITORIA**: cruzar requisitos da spec com codigo (Grep rapido)
6. **VERIFICAR** testes e PRs
7. **PRODUZIR** archive-report.md com veredicto
8. **Se aprovado**: mover para `concluidos/`
9. **ACTUALIZAR** STATUS.md

---

## Gate 6: ARCHIVE - Verificacao de Conclusao

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
- [ ] archive-report.md gerado com veredicto?
- [ ] STATUS.md actualizado (task removida ou marcada como arquivada)?

### Criterios de Aprovacao

```
APROVADO:
- Todos os issues Done/Cancelled
- Requisitos criticos cobertos (mini-auditoria OK)
- archive-report.md gerado

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
| LOC por funcao/metodo | < 45 | > 55 | AP-05 |
| Funcoes por package | < 15 exportadas | > 15 | AP-05 |
| Parametros por funcao | < 5 | > 5 | - |
| Camada correta | 100% | SQL em handler | AP-06 |
| Interfaces com 1 impl | 0 (sem excepcao documentada) | > 0 | AP-01 |
| Padroes por problema | 1 | > 1 | AP-08 |
| Dead code / stubs | 0 | > 0 | AP-07 |

### Seguranca

| Metrica | Target | Red Flag | Anti-Pattern |
|---------|--------|----------|-------------|
| Endpoints com JWT | 100% (publicos) | Endpoint sem auth | - |
| Endpoints com rate limit | 100% (publicos) | Sem limites | - |
| SQL parametrizado | 100% | String concatenation | - |
| Segredos em env | 100% | Hardcoded | - |
| Medidas com enforcement | 100% | Atributo sem handler | AP-03 |
| Sanitizers redundantes | 0 | > 0 | AP-03 |
| Proporcionalidade | 100% | Encriptacao de dados triviais | AP-02 |

### Testes

| Metrica | Target | Red Flag |
|---------|--------|----------|
| Testes de outputs | Sim | Testes de estado interno |
| Cenarios reais | Sim | Edge cases irreais |
| Testes de seguranca | Sim (JWT, rate limit) | Zero testes de auth |

---

*Densare Cloud SDD - Fevereiro 2026*
