# SDD - Specification-Driven Development

## Workflow Densare Cloud

Este documento define o fluxo de trabalho SDD para o ecossistema Cloud da Densare (Plataforma Base + Produtos SaaS).

---

## Principio Fundamental

> **A especificacao e a fonte de verdade, nao o codigo.**

O codigo e um detalhe de implementacao. A spec define o comportamento esperado, restricoes e criterios de aceitacao.

---

## Fluxo SDD

```
PLANEAMENTO (este repo):
+----------+    +----------+    +----------+    +----------+
| REQUEST  | -> | SPECIFY  | -> |   PLAN   | -> |  LINEAR  |
+----------+    +----------+    +----------+    +----------+
     |              |              |               |
     v              v              v               v
 request.md     spec.md        plan.md         issue (handoff)

IMPLEMENTACAO (repos de codigo, outro agente):
+----------+    +--------------+    +----------+    +------------+
|  LINEAR  | -> |  IMPLEMENT   | -> |  CHECK   | -> | END-ISSUE  |  (×N issues por task)
+----------+    +--------------+    +----------+    +------------+
     |               |                  |                |
     v               v                  v                v
 issue + spec     codigo            verificacao     commit+PR+merge
 + plan                                             + fechar Linear

FECHO (este repo — so quando TODOS os issues da task estao Done):
+----------+    +------------------+    +------------+
| /status  | -> |    /close      | -> | concluidos/ |
+----------+    +------------------+    +------------+
     |               |                       |
     v               v                       v
 verificar       mini-auditoria          pasta arquivada
 Linear          + relatorio             + archive-report.md

BUGS (repos de codigo — investigacao + fix no mesmo agente):
+----------+    +--------------+    +----------+    +------------+
|   /bug   | -> |  IMPLEMENT   | -> |  CHECK   | -> | END-ISSUE  |
+----------+    +--------------+    +----------+    +------------+
     |               |                  |                |
     v               v                  v                v
 investiga +      codigo            verificacao     commit+PR+merge
 cria/actualiza   (fix)                             + fechar Linear
 issue Linear

TRIAGEM (requests possivelmente implementados):
+----------+         +--> DONE (fechar)
|  AUDIT   | --------+--> PARCIAL --> /specify (delta) --> ...
+----------+         +--> DIVERGENTE --> decidir
                     +--> NAO INICIADO --> /specify --> ...

Nota: O /plan inclui auditoria automatica como primeiro passo.
      O /audit verifica issues existentes no Linear antes de classificar.
      O /bug vive nos repos de codigo, nao no repo de planeamento.
```

Em cada fase tecnica:
- Ler `ANTI_PATTERNS.md` (obrigatorio em PLAN, IMPLEMENT, CHECK)
- Aplicar Quality Gates de `QUALITY_GATES.md`
- Verificar dependencias cross-module (ver `CLAUDE.md`)
- Investigar codigo existente antes de criar novo

---

### Pre-Pipeline: AUDIT (Triagem)

**Objetivo**: Verificar rapidamente se um request ja esta implementado no codigo, evitando trabalho desnecessario.

**Comando**: `/audit <projecto> <task-name>`

**Quando usar**: Requests que podem ja estar implementados (total ou parcialmente). O `/plan` tambem executa uma auditoria automatica como primeiro passo.

**Classificacoes**:
- **DONE**: 100% implementado → fechar request
- **PARCIAL**: Parte implementada → criar delta-request com o que falta, depois /specify
- **DIVERGENTE**: Implementado mas diferente → decidir com utilizador (ajustar codigo ou request)
- **NAO INICIADO**: Nada implementado → pipeline SDD normal (/specify)

**Regra**: ~15 min por request. E triagem, nao analise profunda.

**Output**: Relatorio breve + `AUDITORIA.md` actualizado

> Comportamento detalhado em `.claude/commands/audit.md`

---

### Fase 0: REQUEST (Utilizador)

**Objetivo**: Capturar o que o utilizador precisa em linguagem simples.

**Comando**: `/request <task-name>`

**Documento**: `request.md`

**Conteudo**:
- O que preciso? (2-3 frases simples)
- Porque preciso? (problema que resolve)
- Como imagino que funcione? (passos do utilizador)
- O que deve aparecer? (resultado esperado)

**Nota**: Este documento NAO e tecnico. Usa linguagem do dia-a-dia. SEM estimativas.

---

### Fase 1: SPECIFY (Equipa Tecnica)

**Objetivo**: Traduzir o pedido em especificacao tecnica.

**Comando**: `/specify <task-name>`

**Input**: `request.md`

**Acoes principais**:
1. Ler `request.md` e `ANTI_PATTERNS.md`
2. Classificar: [PLATFORM], [APP], [ADMIN] ou [SDK]
3. Verificar dependencias cross-module
4. Investigar codigo/servicos existentes (search before create)
5. Criar `spec.md` com requisitos, criterios de aceitacao, edge cases
6. Aplicar Quality Gate SPECIFY

**Output**: `spec.md` aprovada

> Comportamento detalhado em `.claude/commands/specify.md`

---

### Fase 2: PLAN

**Objetivo**: Definir COMO implementar a spec, priorizando modificar codigo existente.

**Comando**: `/plan <task-name>`

**Acoes principais**:
1. Ler spec aprovada e `ANTI_PATTERNS.md`
2. **Investigar codigo existente** — o passo mais importante
3. Decidir: modificar existente > estender > generalizar > criar novo
4. Verificar dependencias cross-module
5. Aplicar Quality Gate PLAN (camadas, interfaces, padroes, LOC)
6. Estimar em pontos Fibonacci (>= 13 obriga a partir)
7. Criar `plan.md`

**Output**: `plan.md` aprovado

> Comportamento detalhado em `.claude/commands/plan.md`

---

### Fase 3: IMPLEMENT

**Objetivo**: Escrever codigo seguindo o plano, modificando existente primeiro.

**Comando**: `/implement <task-name>`

**Acoes principais**:
1. Ler plan.md, spec.md e `ANTI_PATTERNS.md`
2. Verificar pre-condicoes cross-module
3. Implementar passo a passo conforme o plano
4. Modificar ficheiros existentes PRIMEIRO, criar novos DEPOIS
5. Aplicar Quality Gate IMPLEMENT
6. Actualizar `CHANGELOG.md`

**Output**: Codigo implementado

> Comportamento detalhado em `.claude/commands/implement.md`

---

### Fase 4: CHECK

**Objetivo**: Verificar codigo contra spec e anti-patterns.

**Comando**: `/check <task-name>`

**Acoes principais**:
1. Ler spec.md e `ANTI_PATTERNS.md`
2. Verificar cada requisito funcional
3. Contar interfaces vs implementacoes
4. Verificar LOC, duplicacoes, camadas
5. Verificar enforcement de seguranca
6. Produzir relatorio com seccao "Anti-Patterns Detectados"
7. Retrospetiva

**Output**: Relatorio de verificacao

> Comportamento detalhado em `.claude/commands/check.md`

---

### Fase 5: ARCHIVE (Fecho)

**Objetivo**: Verificar conclusao e arquivar task SDD apos todos os issues estarem Done.

**Comando**: `/close <projecto> <task>`

**Pre-condicoes**: TODOS os issues Linear da task devem estar Done (ou Cancelled com justificacao).

**Acoes principais**:
1. Ler spec.md e plan.md da task
2. Identificar todos os issues Linear referenciados
3. Verificar no Linear que TODOS estao Done
4. Mini-auditoria: cruzar requisitos da spec com o codigo (Grep rapido)
5. Verificar que testes existem e PRs estao merged
6. Produzir relatorio com veredicto (APROVADO / REPROVADO)
7. Se aprovado: mover task para `projectos/<proj>/requests/concluidos/<task>/`
8. Gerar `archive-report.md` na pasta destino

**Output**: Relatorio de arquivo + ficheiros movidos para `requests/concluidos/`

> Comportamento detalhado em `.claude/commands/close.md`

---

### Fase 6: MANUTENCAO DE DOCUMENTACAO

**Objetivo**: Manter documentacao limpa e actualizada apos cada ciclo SDD.

**Quando**: Apos cada `/close` e periodicamente.

**Accoes**:
1. **REMOVER** ficheiros temporarios e irrelevantes (drafts abandonados, specs canceladas)
2. **ACTUALIZAR** documentos que foram afectados pela implementacao:
   - `CLAUDE.md` se a arquitectura mudou
   - `QUALITY_GATES.md` se surgiram novos padroes ou regras
   - `ANTI_PATTERNS.md` se foram descobertos novos anti-patterns
   - `CHANGELOG.md` com as alteracoes feitas
3. **VERIFICAR** consistencia entre ficheiros:
   - Specs reflectem o que foi implementado?
   - Referencias entre ficheiros estao correctas?
   - Nao ha informacao duplicada entre ficheiros?

**Principio**: Cada ficheiro deve ter UMA fonte de verdade. Se a mesma informacao aparece em 2+ ficheiros, escolher um como fonte e os outros referenciam.

---

### Tracking: /status

**Objetivo**: Visao global do estado de todas as tasks de um projecto.

**Comando**: `/status [projecto] [task]`

**Tres modos**:
- Sem argumentos: visao global de todos os projectos
- So projecto: todas as tasks do projecto com estado Linear actualizado
- Projecto + task: detalhe de uma task individual

**STATUS.md**: Cada projecto tem um `projectos/<projecto>/STATUS.md` actualizado automaticamente pelo `/status`. E um snapshot (nao um log) com estado SDD + progresso Linear de cada task.

> **Nota**: Tasks antigas podem ainda estar em `specs/`. Novos requests usam o pipeline `requests/`.

> Comportamento detalhado em `.claude/commands/status.md`

---

## Estrutura de Pastas

```
cloud/
+-- CLAUDE.md                           # Regras, classificacao, convencoes
+-- sdd/
|   +-- ANTI_PATTERNS.md               # Anti-patterns a evitar (LEITURA OBRIGATORIA)
|   +-- WORKFLOW.md                     # Este documento
|   +-- QUALITY_GATES.md               # Verificacoes de qualidade por fase
|   +-- CHANGELOG.md                   # Historico de alteracoes
|   +-- _templates/                    # Templates
|       +-- request.md
|       +-- spec.md
|       +-- plan.md
+-- .claude/
|   +-- commands/                      # Slash commands (fonte de verdade do comportamento)
|       +-- request.md
|       +-- specify.md
|       +-- plan.md
|       +-- implement.md
|       +-- check.md
|       +-- archive.md
|       +-- audit.md
|       +-- status.md
+-- projectos/
|   +-- README.md                     # Indice dos 16 produtos
|   +-- plataforma-base/
|   |   +-- STATUS.md                 # Snapshot do estado (actualizado por /status)
|   |   +-- AUDITORIA.md              # Resultados de triagem (/audit)
|   |   +-- specs/                    # Specs de referencia (per-service)
|   |   +-- requests/                 # Pipeline SDD
|   |   |   +-- em-analise/
|   |   |   +-- aprovados/
|   |   |   +-- em-implementacao/
|   |   |   +-- concluidos/
|   |   |   +-- rejeitados/
|   |   +-- linear/                   # Snapshots de issues
|   |   +-- concorrencia/
|   |   +-- estudos/
|   |   +-- ui/
|   +-- 2snip/                        # Mesma estrutura
|   +-- 2send/                        # Mesma estrutura
|   +-- ai-product-manager/          # Mesma estrutura (minimalista)
+-- referencia/
|   +-- arquitectura/                 # 01-08_*.md (docs de referencia cruzada)
|   +-- roadmap/                      # Resumos, roadmap, ecossistema
+-- backlog/                          # Modulos futuros (estudos)
+-- arquivo/                          # Material arquivado
```

### Documentos por Audiencia

| Documento | Quem escreve | Quem le | Linguagem |
|-----------|--------------|---------|-----------|
| `request.md` | Utilizador | Todos | Simples, dia-a-dia |
| `spec.md` | Equipa tecnica | Equipa | Tecnica |
| `plan.md` | Equipa tecnica | Equipa | Tecnica |

### Fontes de Verdade

| Assunto | Ficheiro | NAO duplicar em |
|---------|----------|-----------------|
| Comportamento dos slash commands | `.claude/commands/*.md` | CLAUDE.md, WORKFLOW.md |
| Anti-patterns e limites LOC | `sdd/ANTI_PATTERNS.md` | QUALITY_GATES.md (referenciar apenas) |
| Quality gates e checklists | `sdd/QUALITY_GATES.md` | CLAUDE.md |
| Classificacao e convencoes | `CLAUDE.md` | WORKFLOW.md |
| Fluxo e fases SDD | `sdd/WORKFLOW.md` (este ficheiro) | CLAUDE.md |

---

## Slash Commands

| Comando | Descricao | Fase |
|---------|-----------|------|
| `/request <task>` | Criar pedido de funcionalidade | REQUEST |
| `/specify <task>` | Traduzir pedido em spec tecnica | SPECIFY |
| `/plan <task>` | Gerar plano de implementacao | PLAN |
| `/implement <task>` | Implementar seguindo o plano | IMPLEMENT |
| `/check <task>` | Verificar codigo contra spec | CHECK |
| `/close <proj> <task>` | Mini-auditoria + mover para concluido | ARCHIVE |
| `/audit <proj> [task]` | Triagem de request vs codigo existente | AUDIT (pre-pipeline) |
| `/status [proj] [task]` | Visao global ou detalhe + actualizar STATUS.md | Qualquer |

> Comportamento detalhado de cada comando em `.claude/commands/`

---

## Regras Criticas por Fase

### Durante SPECIFY
- [ ] Ler `ANTI_PATTERNS.md`
- [ ] NAO discutir implementacao
- [ ] Investigar codigo/servicos existentes
- [ ] Classificar: [PLATFORM], [APP], [ADMIN] ou [SDK]
- [ ] Verificar dependencias cross-module
- [ ] Definir criterios de aceitacao (Given/When/Then)

### Durante PLAN
- [ ] **Auditoria automatica** — verificar se feature ja existe no codigo (passo 0 obrigatorio)
- [ ] Ler `ANTI_PATTERNS.md`
- [ ] **Investigar codigo existente** — modificar > estender > criar
- [ ] NAO escrever codigo
- [ ] Validar camadas (handler -> service -> repository)
- [ ] Justificar cada ficheiro novo (porque nao modificar existente?)
- [ ] Verificar dependencias cross-module
- [ ] Estimar (Fibonacci). >= 13 pontos: obrigatorio partir

### Durante IMPLEMENT
- [ ] Ler `ANTI_PATTERNS.md`
- [ ] Seguir o plano aprovado
- [ ] Modificar ficheiros existentes PRIMEIRO
- [ ] NAO adicionar features nao especificadas
- [ ] Verificar limites LOC (ficheiro < 300, funcao < 30)
- [ ] Actualizar CHANGELOG.md

### Durante CHECK
- [ ] Ler `ANTI_PATTERNS.md`
- [ ] Contar interfaces vs implementacoes (AP-01)
- [ ] Verificar LOC dos ficheiros (AP-05)
- [ ] Verificar enforcement de seguranca (AP-03)
- [ ] Verificar duplicacoes (AP-04)
- [ ] **Verificar documentacao**: actualizada, limpa, sem duplicacoes?

---

## Nomenclatura de Tasks

**Formato**: `<servico>-<funcionalidade>` em kebab-case

**Exemplos**: `auth-jwt-validation`, `license-keep-alive`, `gateway-rate-limiting`, `sdk-offline-mode`

---

## Estados de uma Task

```
DRAFT -> SPECIFIED -> PLANNED -> IN_PROGRESS -> REVIEW -> DONE -> READY TO ARCHIVE -> ARCHIVED
```

- **DRAFT**: spec.md em elaboracao
- **SPECIFIED**: spec.md aprovada
- **PLANNED**: plan.md aprovado + issues Linear criados
- **IN_PROGRESS**: implementacao em curso (algum issue In Progress)
- **REVIEW**: aguarda verificacao (/check)
- **DONE**: todos os issues Linear Done
- **READY TO ARCHIVE**: todos Done, pronto para `/close`
- **ARCHIVED**: mini-auditoria aprovada, ficheiros em `requests/concluidos/`

O `/status` determina o estado automaticamente com base nos ficheiros locais e no Linear.

---

## Agentes Especializados

| Fase | Agente | Quando Usar |
|------|--------|-------------|
| SPECIFY | `requirements-analyst` | Requisitos complexos, dominios desconhecidos |
| PLAN | `software-architect` | Decisoes arquitecturais, novos servicos |
| IMPLEMENT | `refactoring-specialist` | Reestruturacao de codigo existente |
| CHECK | `code-reviewer` | Revisao de qualidade pos-implementacao |
| DEBUG | `bug-debugger` | Problemas identificados no /check |
| TESTES | `test-engineer` | Estrategia e criacao de testes |
| DOCS | `documentation-writer` | APIs, runbooks |
| INTEGRACAO | `integration-specialist` | APIs externas (ifthenpay, Brevo, Creem, Paddle) |

**Usar agente quando**: task envolve 3+ ficheiros, decisoes arquitecturais, dominio complexo, API externa.

---

## Escala de Estimativas (Fibonacci)

### Issues (Linear)

| Pontos | Duracao | Uso |
|--------|---------|-----|
| 1 | ~30min | Muito simples |
| 2 | ~1h | Tipico |
| 3 | ~2h | Medio |
| 5 | ~4h | Complexo |
| 8 | ~8h | Grande |
| 13 | ~16h | Partir em sub-issues |
| 21 | ~32h | DEVE ser partido |

**Regras**: Maioria 1-2 pontos. > 8: avaliar partir. >= 13: obrigatorio partir.

### Planeamento (Requests SDD / Roadmaps)

> Para estimativas de alto nivel em requests, roadmaps e planeamento de produto. Estes valores **nao se usam no Linear** — servem para dimensionar features completas antes da decomposicao em issues.

| Pontos | Duracao | Uso |
|--------|---------|-----|
| 34 | ~55h | Feature muito grande — obrigatorio partir em sub-requests |
| 55 | ~90h | Epico — multiplos sub-requests |
| 89 | ~145h | Mega-epico — modulo completo ou feature transversal |

---

## Referencias

- Regras e convencoes: `CLAUDE.md`
- Anti-patterns: `sdd/ANTI_PATTERNS.md`
- Quality gates: `sdd/QUALITY_GATES.md`
- Arquitectura: `referencia/arquitectura/`
- Roadmap: `referencia/roadmap/`
- Specs: `projectos/<projecto>/specs/`
- Requests: `projectos/<projecto>/requests/`

---

*Densare Cloud SDD - Fevereiro 2026*
