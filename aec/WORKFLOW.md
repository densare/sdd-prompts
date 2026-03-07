# SDD - Specification-Driven Development

## Workflow Densare AEC

Este documento define o fluxo de trabalho SDD para o ecossistema AEC da Densare (DenStudio, DenTherm, DenSim).

---

## Principio Fundamental

> **A especificacao e a fonte de verdade, nao o codigo.**

O codigo e um detalhe de implementacao. A spec define o comportamento esperado, restricoes e criterios de aceitacao.

---

## Papeis e Responsabilidades

| Papel | Quem | Responsabilidades |
|-------|------|-------------------|
| **Product Owner** | Utilizador / Gestor de projecto | Cria requests (`/sdd-request`), aprova specs, valida prioridades, decide scope |
| **Agente SDD** | Claude (AI) | Especifica (`/sdd-specify`), planeia (`/sdd-plan`), audita (`/sdd-audit`), verifica (`/sdd-check`), arquiva (`/sdd-archive`) |
| **Implementador** | Claude (AI) no repo de codigo | Implementa (`/sdd-implement`) seguindo plano aprovado |
| **Revisor** | Utilizador + `/sdd-check` | Aprova PRs, valida implementacao contra spec |

### Fluxo de Aprovacoes

```
REQUEST  → Product Owner cria
SPECIFY  → Agente SDD cria spec → Product Owner APROVA (ou pede alteracoes)
PLAN     → Agente SDD cria plan → Product Owner APROVA (ou pede alteracoes)
IMPLEMENT → Implementador executa (sem aprovacao intermedia)
CHECK    → Agente SDD verifica → Product Owner VALIDA resultado
CLOSE    → Agente SDD verifica → automatico se quality gates passam
ARCHIVE  → Agente SDD gera plano de teste → Product Owner valida
```

**Regra**: Nenhuma spec ou plano avanca sem aprovacao explicita do Product Owner. O Agente SDD propoe, o Product Owner decide.

---

## Fluxo SDD

```
PLANEAMENTO (este repo):
+--------------+    +--------------+    +----------+    +----------+
| SDD-REQUEST  | -> | SDD-SPECIFY  | -> | SDD-PLAN | -> |  LINEAR  |
+--------------+    +--------------+    +----------+    +----------+
      |                  |                  |               |
      v                  v                  v               v
  request.md          spec.md           plan.md       issue (handoff)

IMPLEMENTACAO (repos de codigo — denstudio A/B/C, dentherm A/B/C):
+----------+    +----------------+    +-----------+    +---------+    +------------+
|  LINEAR  | -> | SDD-IMPLEMENT  | -> | SDD-CHECK | -> | SDD-FIX | -> | END-ISSUE  |
+----------+    +----------------+    +-----------+    +---------+    +------------+
     |                 |                   |                |              |
     v                 v                   v                v              v
 issue + spec       codigo          check-report.md   corrige +      commit+PR+merge
 + plan          (Kimi/outro)       (Claude/revisor)  apaga report   + fechar Linear
                                         |                |
                                         v                v
                                    REJECTED? ---------> /sdd-fix --> /sdd-check (loop)
                                    APPROVED? ---------> /end-issue
                                    APPROVED + DEFERRALS? --> issues Linear criados --> /end-issue

FECHO (este repo — so quando TODOS os issues da task estao Done):
+--------------+    +------------------+    +------------+    +--------------+    +------------+
| /sdd-status  | -> |  /sdd-close      | -> | concluido/ | -> | /sdd-archive | -> | arquivados/|
+--------------+    +------------------+    +------------+    +--------------+    +------------+
     |               |                       |                     |                   |
     v               v                       v                     v                   v
 verificar       mini-auditoria          pasta concluida       plano de teste       pasta final
 Linear          + relatorio             + close-report.md     + user testa         + accept-report.md
                                                              + reporta defeitos

BUGS (repos de codigo — investigacao e fix podem ser modelos diferentes):
+----------+    +----------------+    +-----------+    +------------+
| /sdd-bug | -> | SDD-IMPLEMENT  | -> | SDD-CHECK | -> | END-ISSUE  |
+----------+    +----------------+    +-----------+    +------------+
     |                 |                   |                |
     v                 v                   v                v
 investiga +      codigo (fix)       check-report.md   commit+PR+merge
 bug-report.md    (le bug-report,    (Claude revê)     + fechar Linear
 (Claude)         apaga-o)
                  (MiniMax/Kimi)

 Deteccao de contexto pelo /sdd-bug:
   - /sdd-bug DT-XXX         → issue existente, investigar e planear fix
   - /sdd-bug "descricao"    → verificar branch:
       em feature/XX-NNN     → bug no trabalho actual, comentar no issue, sem novo issue
       em main               → bug novo, criar issue Linear

TRIAGEM (requests possivelmente implementados):
+-----------+         +--> DONE (fechar)
| SDD-AUDIT | --------+--> PARCIAL --> /sdd-specify (delta) --> ...
+-----------+         +--> DIVERGENTE --> decidir
                      +--> NAO INICIADO --> /sdd-specify --> ...

Nota: O /sdd-plan inclui auditoria automatica como primeiro passo.
      O /sdd-audit verifica issues existentes no Linear antes de classificar.
      O /sdd-bug vive nos repos de codigo. Detecta contexto pelo branch git.
```

Em cada fase tecnica:
- Ler `ANTI_PATTERNS.md` (obrigatorio em PLAN, IMPLEMENT, CHECK)
- Aplicar Quality Gates de `QUALITY_GATES.md`
- Verificar dependencias cross-module (ver `CLAUDE.md`)
- Investigar codigo existente antes de criar novo

---

### Pre-Pipeline: AUDIT (Triagem)

**Objetivo**: Verificar rapidamente se um request ja esta implementado no codigo, evitando trabalho desnecessario.

**Comando**: `/sdd-audit <projecto> <request-id>`

**Quando usar**: Requests que podem ja estar implementados (total ou parcialmente).

**Classificacoes**:
- **DONE**: 100% implementado → fechar request
- **PARCIAL**: Parte implementada → criar delta-request com o que falta, depois /sdd-specify
- **DIVERGENTE**: Implementado mas diferente → decidir com utilizador (ajustar codigo ou request)
- **NAO INICIADO**: Nada implementado → pipeline SDD normal (/sdd-specify)

**Regra**: ~15 min por request. E triagem, nao analise profunda.

**Output**: Relatorio breve + `AUDITORIA.md` actualizado

> Comportamento detalhado em `.claude/commands/sdd-audit.md`

---

### Fase 0: REQUEST (Utilizador)

**Objetivo**: Capturar o que o utilizador precisa em linguagem simples.

**Comando**: `/sdd-request <task-name>`

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

**Comando**: `/sdd-specify <task-name>`

**Input**: `request.md`

**Acoes principais**:
1. Ler `request.md` e `ANTI_PATTERNS.md`
2. Classificar: [CORE], [THERMAL], [SIM] ou [APP]
3. Verificar dependencias cross-module
4. Investigar codigo/classes existentes (search before create)
5. Criar `spec.md` com requisitos, criterios de aceitacao, edge cases
6. Aplicar Quality Gate SPECIFY

**Output**: `spec.md` aprovada

> Comportamento detalhado em `.claude/commands/sdd-specify.md`

---

### Fase 2: PLAN

**Objetivo**: Definir COMO implementar a spec, priorizando modificar codigo existente.

**Comando**: `/sdd-plan <task-name>`

**Acoes principais**:
1. Ler spec aprovada e `ANTI_PATTERNS.md`
2. **Investigar codigo existente** — o passo mais importante
3. Decidir: modificar existente > estender > generalizar > criar novo
4. Verificar dependencias cross-module
5. Aplicar Quality Gate PLAN (camadas, interfaces, padroes, LOC)
6. Estimar em pontos Fibonacci (>= 13 obriga a partir)
7. Criar `plan.md`

**Output**: `plan.md` aprovado

> Comportamento detalhado em `.claude/commands/sdd-plan.md`

---

### Fase 3: IMPLEMENT

**Objetivo**: Escrever codigo seguindo o plano, modificando existente primeiro.

**Comando**: `/sdd-implement <task-name>`

**Acoes principais**:
1. Ler plan.md, spec.md e `ANTI_PATTERNS.md`
2. Verificar pre-condicoes cross-module
3. Implementar passo a passo conforme o plano
4. Modificar ficheiros existentes PRIMEIRO, criar novos DEPOIS
5. Aplicar Quality Gate IMPLEMENT
6. Actualizar `CHANGELOG.md`

**Output**: Codigo implementado

> Comportamento detalhado em `.claude/commands/sdd-implement.md`

---

### Fase 4: CHECK

**Objetivo**: Verificar codigo contra spec e anti-patterns.

**Comando**: `/sdd-check <task-name>`

**Regra de Revisao Independente**: O `/sdd-check` NUNCA deve correr na mesma sessao que o `/sdd-implement`. Usar sessao nova ou repo alternativo (A/B) para garantir revisao sem vies do implementador.

**Acoes principais**:
1. Ler spec.md e `ANTI_PATTERNS.md`
2. Verificar cada requisito funcional
3. Contar interfaces vs implementacoes
4. Verificar LOC, duplicacoes, camadas
5. Verificar enforcement de seguranca
6. Produzir relatorio com seccao "Anti-Patterns Detectados"
7. Retrospetiva

**Output**: Relatorio de verificacao

> Comportamento detalhado em `.claude/commands/sdd-check.md`

---

### Fase 5: CLOSE (Fecho Tecnico)

**Objetivo**: Verificar conclusao tecnica e fechar task SDD apos todos os issues estarem Done.

**Comando**: `/sdd-close <projecto> <task>`

**Pre-condicoes**: TODOS os issues Linear da task devem estar Done (ou Cancelled com justificacao).

**Acoes principais**:
1. Ler spec.md e plan.md da task
2. Identificar todos os issues Linear referenciados
3. Verificar no Linear que TODOS estao Done
4. Mini-auditoria: cruzar requisitos da spec com o codigo (Grep rapido)
5. Verificar que testes existem e PRs estao merged
6. Produzir relatorio com veredicto (APROVADO / REPROVADO)
7. Se aprovado: mover `projectos/<proj>/requests/em-implementacao/<task>/` para `projectos/<proj>/requests/concluidos/<task>/`
8. Gerar `close-report.md` na pasta destino

**Output**: Relatorio de fecho + ficheiros movidos para `concluidos/`

> Comportamento detalhado em `.claude/commands/sdd-close.md`

---

### Fase 6: ARCHIVE (Aceitacao pelo Utilizador + Arquivo)

**Objetivo**: O utilizador valida manualmente que a implementacao corresponde ao pedido original. Esta e a fase final — o Product Owner testa e aceita (ou rejeita) a funcionalidade, e a task e arquivada.

**Comando**: `/sdd-archive <projecto> <task>`

**Pre-condicoes**: Task em `concluidos/` (ja passou por `/sdd-close`).

**Acoes principais**:
1. Ler request.md e spec.md da task
2. Gerar resumo em linguagem simples do que foi implementado
3. Descrever onde encontrar a funcionalidade na aplicacao
4. Descrever o que o utilizador deve ver (UI, layout, comportamento)
5. Gerar cenarios de teste passo-a-passo para CADA requisito funcional (RF-XX)
6. Gerar exemplo completo end-to-end com dados realistas
7. Gerar `accept-report.md` na pasta da task
8. Apresentar plano ao utilizador e recolher veredicto

**Veredictos**:
- **ACEITE**: Mover para `arquivados/`. Funcionalidade fechada.
- **ACEITE COM RESERVAS**: Mover para `arquivados/`. Defeitos menores registados como issues Linear.
- **REJEITADO**: Fica em `concluidos/`. Defeitos criticos registados como issues. Apos correcao, re-testar com novo `/sdd-archive`.

**Output**: Plano de teste + `accept-report.md` preenchido pelo utilizador

> Comportamento detalhado em `.claude/commands/sdd-archive.md`

---

### Fase 7: MANUTENCAO DE DOCUMENTACAO

**Objetivo**: Manter documentacao limpa e actualizada apos cada ciclo SDD.

**Quando**: Apos cada `/sdd-close` e periodicamente.

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

### Tracking: /sdd-status

**Objetivo**: Visao global do estado de todas as tasks de um projecto.

**Comando**: `/sdd-status [projecto] [task]`

**Tres modos**:
- Sem argumentos: visao global de todos os projectos
- So projecto: todas as tasks do projecto com estado Linear actualizado
- Projecto + task: detalhe de uma task individual

**STATUS.md**: Cada projecto tem um `projectos/<projecto>/STATUS.md` actualizado automaticamente pelo `/sdd-status`. E um snapshot (nao um log) com estado SDD + progresso Linear de cada task.

**README.md de requests**: Cada projecto tem `projectos/<projecto>/requests/README.md` **regenerado automaticamente** pelo `/sdd-status` (Modo 2) a partir do estado real dos ficheiros. Nunca editado manualmente — a fonte de verdade sao os ficheiros nas subpastas.

> Comportamento detalhado em `.claude/commands/sdd-status.md`

---

## Estrutura de Pastas

```
aec/
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
|       +-- sdd-request.md
|       +-- sdd-specify.md
|       +-- sdd-plan.md
|       +-- sdd-implement.md
|       +-- sdd-check.md
|       +-- sdd-close.md
|       +-- sdd-archive.md
|       +-- sdd-audit.md
|       +-- sdd-validate.md
|       +-- sdd-status.md
+-- projectos/
    +-- <projecto>/
        +-- STATUS.md                  # Snapshot do estado (actualizado por /sdd-status)
        +-- AUDITORIA.md              # Resultados de triagem (actualizado por /sdd-audit)
        +-- specs/                     # Specs gerais de referencia (00-06_*.md)
        +-- requests/
            +-- README.md             # Resumo dos requests (contadores, tabelas)
            +-- em-analise/           # Ficheiros simples .md (pendentes de revisao)
            |   +-- C1-dados-gerais.md
            +-- aprovados/            # Ficheiros simples .md OU pastas (com spec)
            |   +-- B2-identificacao.md
            +-- em-implementacao/     # Pastas com todos os artefactos
            |   +-- A8-selecionar-modulo/
            |       +-- request.md
            |       +-- spec.md
            |       +-- plan.md
            |       +-- issues.md
            +-- concluidos/           # Pastas concluidas (todos issues Done, aguarda aceitacao)
            |   +-- B1-criar-processo/
            |       +-- request.md
            |       +-- spec.md
            |       +-- close-report.md
            +-- arquivados/           # Pastas aceites pelo utilizador (estado FINAL)
            |   +-- A1-feature-x/
            |       +-- request.md
            |       +-- spec.md
            |       +-- close-report.md
            |       +-- accept-report.md
            +-- rejeitados/           # Ficheiros descartados
```

> **Convencao**: Requests simples (sem spec) sao ficheiros `.md`. Requests com spec/plan sao **pastas** com todos os artefactos juntos.

### Documentos por Audiencia

| Documento | Quem escreve | Quem le | Linguagem |
|-----------|--------------|---------|-----------|
| `request.md` | Utilizador | Todos | Simples, dia-a-dia |
| `spec.md` | Equipa tecnica | Equipa | Tecnica |
| `plan.md` | Equipa tecnica | Equipa | Tecnica |
| `accept-report.md` | Agente SDD (gera) + Utilizador (preenche) | Todos | Simples, dia-a-dia |

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
| `/sdd-request <task>` | Criar pedido de funcionalidade | REQUEST |
| `/sdd-specify <task>` | Traduzir pedido em spec tecnica | SPECIFY |
| `/sdd-plan <task>` | Gerar plano de implementacao | PLAN |
| `/sdd-implement <task>` | Implementar seguindo o plano | IMPLEMENT |
| `/sdd-check <task>` | Verificar codigo contra spec, escrever check-report.md | CHECK |
| `/sdd-fix <task>` | Aplicar correccoes do check-report, apagar report | FIX |
| `/sdd-bug <issue ou texto>` | Investigar bug, escrever bug-report.md, criar/actualizar issue | BUG |
| `/sdd-close <proj> <task>` | Mini-auditoria + mover para concluidos | CLOSE |
| `/sdd-archive <proj> <task>` | Plano de teste manual + aceitacao + mover para arquivados | ARCHIVE |
| `/sdd-audit <proj> [id]` | Triagem de request vs codigo existente | AUDIT (pre-pipeline) |
| `/sdd-status [proj] [task]` | Visao global ou detalhe + actualizar STATUS.md | Qualquer |

> Comportamento detalhado de cada comando em `.claude/commands/`

---

## Regras Criticas por Fase

### Durante SPECIFY (/sdd-specify)
- [ ] Ler `ANTI_PATTERNS.md`
- [ ] NAO discutir implementacao
- [ ] Investigar codigo/classes existentes
- [ ] Classificar: [CORE], [THERMAL], [SIM] ou [APP]
- [ ] Verificar dependencias cross-module
- [ ] Definir criterios de aceitacao (Given/When/Then)

### Durante PLAN (/sdd-plan)
- [ ] **Auditoria automatica** — verificar se feature ja existe no codigo (passo 0 obrigatorio)
- [ ] Ler `ANTI_PATTERNS.md`
- [ ] **Investigar codigo existente** — modificar > estender > criar
- [ ] NAO escrever codigo
- [ ] Validar camadas (Presentation -> Application -> Domain -> Infrastructure)
- [ ] Justificar cada ficheiro novo (porque nao modificar existente?)
- [ ] **Grafo de dependencias** — identificar passos paralelos e sugerir repos A/B
- [ ] **Riscos e rollback** — documentar riscos, mitigacoes e plano de reversao
- [ ] Verificar dependencias cross-module
- [ ] Estimar (Fibonacci). >= 13 pontos: obrigatorio partir

### Durante IMPLEMENT (/sdd-implement)
- [ ] Ler `ANTI_PATTERNS.md`
- [ ] Seguir o plano aprovado
- [ ] **RED/GREEN/VERIFY** para cada RF-XX: escrever teste que falha (RED), implementar (GREEN), correr testes (VERIFY)
- [ ] Respeitar **grafo de dependencias** do plan.md (paralelizacao)
- [ ] Modificar ficheiros existentes PRIMEIRO
- [ ] NAO adicionar features nao especificadas
- [ ] Verificar limites LOC (ficheiro < 500, metodo < 45)
- [ ] Actualizar CHANGELOG.md

### Durante CHECK (/sdd-check)
- [ ] **Revisao independente** — sessao diferente do /sdd-implement
- [ ] Ler `ANTI_PATTERNS.md`
- [ ] Contar interfaces vs implementacoes (AP-01)
- [ ] Verificar LOC dos ficheiros (AP-05)
- [ ] Verificar enforcement de seguranca (AP-03)
- [ ] Verificar duplicacoes (AP-04)
- [ ] **Verificar documentacao**: actualizada, limpa, sem duplicacoes?
- [ ] **Escrever `check-report.md`** na raiz do repo de codigo (formato tabular PASS/FAIL)
- [ ] Veredicto: APPROVED / APPROVED WITH DEFERRALS / REJECTED
- [ ] Se APPROVED WITH DEFERRALS: criar issues Linear para cada deferral

### Durante FIX (/sdd-fix)
- [ ] Ler `check-report.md` da raiz do repo
- [ ] Listar todos os FAIL do relatorio
- [ ] Corrigir cada FAIL (RED/GREEN/VERIFY para requisitos)
- [ ] Build e testes passam apos correccoes
- [ ] **Apagar `check-report.md`** — sinaliza "pronto para re-review"
- [ ] Commit e push das correccoes

### Durante END-ISSUE (/end-issue)
- [ ] **Verificar que `check-report.md` existe com APPROVED ou APPROVED WITH DEFERRALS**
- [ ] Se deferrals: verificar que todos tem issue Linear associado
- [ ] Bloquear se nao existe, REJECTED, ou deferrals sem issue

---

## Nomenclatura de Tasks

**Formato**: `<modulo>-<funcionalidade>` em kebab-case

**Exemplos**: `thermal-zone-calculation`, `core-settings-migration`, `app-navigation-tree`

---

## Estados de uma Task

```
DRAFT -> SPECIFIED -> PLANNED -> IN_PROGRESS -> REVIEW -> FIXING -> DONE -> READY TO CLOSE -> CLOSED -> ARCHIVING -> ARCHIVED
                                                  ^        |        |                                           |
                                                  |        v        |                                           v
                                                  +--- REJECTED ----+                                     REJECTED (bugs)
                                                                                                              |
                                                                                                              v
                                                                                                         fix + re-test
```

- **DRAFT**: spec.md em elaboracao
- **SPECIFIED**: spec.md aprovada
- **PLANNED**: plan.md aprovado + issues Linear criados
- **IN_PROGRESS**: implementacao em curso (algum issue In Progress)
- **REVIEW**: aguarda verificacao (/sdd-check) — check-report.md a ser escrito
- **FIXING**: check-report.md REJECTED — implementador a corrigir (/sdd-fix)
- **DONE**: check-report.md APPROVED + todos os issues Linear Done
- **READY TO CLOSE**: todos Done, pronto para `/sdd-close`
- **CLOSED**: mini-auditoria aprovada, ficheiros em `concluidos/`
- **ARCHIVING**: plano de aceitacao gerado, utilizador a testar (`/sdd-archive`)
- **ARCHIVED**: utilizador validou, ficheiros em `arquivados/` — **estado final**

O `/sdd-status` determina o estado automaticamente com base nos ficheiros locais e no Linear.

---

## Agentes Especializados

| Fase | Agente | Quando Usar |
|------|--------|-------------|
| SPECIFY | `requirements-analyst` | Requisitos complexos, dominios desconhecidos |
| PLAN | `software-architect` | Decisoes arquitecturais, novos modulos |
| IMPLEMENT | `refactoring-specialist` | Reestruturacao de codigo existente |
| CHECK | `code-reviewer` | Revisao de qualidade pos-implementacao |
| DEBUG | `bug-debugger` | Problemas identificados no /check |
| TESTES | `test-engineer` | Estrategia e criacao de testes |
| DOCS | `documentation-writer` | APIs, user guides |
| INTEGRACAO | `integration-specialist` | APIs externas (Cloud Densare) |

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

## Fast-Track (Alteracoes Simples)

> Para alteracoes pequenas (estimativa <= 2 pontos Fibonacci, ~1h) que nao justificam o pipeline SDD completo.

### Quando Usar Fast-Track

**TODOS os criterios devem ser verdadeiros**:
- [ ] Estimativa <= 2 pontos Fibonacci (~1h de trabalho)
- [ ] Toca apenas 1 repositorio
- [ ] Nao cria novos namespaces, projectos ou modulos
- [ ] Sem decisoes arquitecturais (usa padroes ja existentes)
- [ ] Sem dependencias cross-module
- [ ] O que fazer e claro (sem ambiguidade no pedido)

### Quando NAO Usar Fast-Track

- Task que cria novos padroes ou estabelece precedentes
- Task que toca dominio regulamentar (REH, formulas de calculo)
- Task que envolve seguranca ou dados sensiveis
- Task com mais de 5 ficheiros afetados
- Qualquer duvida → usar pipeline completo

### Fluxo Fast-Track

```
REQUEST (simplificado) → IMPLEMENT → CHECK (simplificado)
```

1. **REQUEST simplificado**: Descricao breve (3-5 frases) no request.md. Sem template completo.
2. **Skip SPECIFY e PLAN**: Nao e necessario spec.md nem plan.md formais.
3. **IMPLEMENT**: Seguir as regras normais (anti-patterns, LOC limits, search before create).
4. **CHECK simplificado**: Verificacao rapida — codigo funciona? Anti-patterns? LOC OK?

### Documentacao Fast-Track

O request fast-track vive como ficheiro simples em `em-analise/` com tag `[FAST-TRACK]` no topo:

```markdown
# [ID]. [Nome] [FAST-TRACK]

**O que**: [descricao breve]
**Porque**: [motivacao]
**Ficheiros**: [lista de ficheiros a alterar]
**Issue Linear**: [ID] (criar antes de implementar)
```

Apos implementacao, o ficheiro move directamente para `concluidos/` (sem converter em pasta).

### Exemplos de Fast-Track

| Sim (Fast-Track) | Nao (Pipeline Completo) |
|-------------------|------------------------|
| Adicionar campo a ViewModel existente | Criar novo ViewModel + View |
| Corrigir validacao num campo | Redesenhar sistema de validacao |
| Ajustar layout de painel existente | Criar novo painel |
| Adicionar coluna a tabela existente | Criar nova tabela/entidade |
| Fix de bug com causa conhecida | Bug com causa desconhecida |

---

## Exemplos Preenchidos

Templates preenchidos com exemplos reais estao disponiveis em `sdd/_templates/_exemplos/`:

| Template | Exemplo | Baseado em |
|----------|---------|-----------|
| `request.md` | `_exemplos/request-exemplo.md` | C2 - Areas e Volumes |
| `spec.md` | `_exemplos/spec-exemplo.md` | C2 - Areas e Volumes |
| `plan.md` | `_exemplos/plan-exemplo.md` | C2 - Areas e Volumes |

Usar como referencia ao criar novos artefactos SDD.

---

## Referencias

- Regras e convencoes: `CLAUDE.md`
- Anti-patterns: `sdd/ANTI_PATTERNS.md`
- Quality gates: `sdd/QUALITY_GATES.md`

---

*Densare AEC SDD - Fevereiro 2026*
