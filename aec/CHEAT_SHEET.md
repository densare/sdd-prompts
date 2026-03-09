# SDD Cheat Sheet

> Referencia rapida de todos os comandos SDD. Para detalhes, ver `.claude/commands/<comando>.md`.

---

## Ciclo de Vida de um Request

```
PLANEAMENTO (repo aec/)
  em-analise/ ──> aprovados/ ──> (pasta) ──> em-implementacao/ ──> concluidos/
       │              │              │               │                  │
   /sdd-request   aprovar      /sdd-specify     /sdd-plan        /sdd-close
   (ficheiro)    (manual)     (cria pasta      (plan + issues     (mini-audit
    simples)                  + spec.md)       + move pasta)      + mover)

IMPLEMENTACAO (repos de codigo — denstudio A/B/C, dentherm A/B/C)
  issue Linear ──> codigo ──> verificacao ──> commit + PR + merge
       │              │            │                │
   /sdd-implement  seguir     /sdd-check       /end-issue
                   plano

BUGS (repos de codigo)
  descricao/ID ──> investiga ──> fix ──> verificacao ──> commit + PR
       │               │          │          │              │
     /fix-bug      analise    /sdd-implement /sdd-check  /end-issue
```

---

## Comandos — Planeamento (repo `aec/`)

| Comando | Quando | O que faz |
|---------|--------|-----------|
| `/sdd-request <proj> <ID>-<nome>` | Novo pedido | Cria `.md` em `em-analise/` |
| `/sdd-audit <proj> <ID>` | Triagem | Verifica se ja esta implementado no codigo |
| `/sdd-audit <proj>` | Visao geral | Lista requests e estado de auditoria |
| `/sdd-specify <proj> <ID>` | Apos aprovacao | Converte ficheiro em pasta + cria `spec.md` |
| `/sdd-plan <proj> <ID>` | Apos spec | Cria `plan.md` + `issues.md` + issue Linear + move para `em-implementacao/` |
| `/sdd-close <proj> <ID>` | Todos issues Done | Mini-auditoria + move para `concluidos/` |
| `/sdd-archive <proj> <ID>` | Apos close | User testa + aceita + move para `arquivados/` |
| `/sdd-status` | Qualquer altura | Visao global de todos os projectos |
| `/sdd-status <proj>` | Qualquer altura | Todas as tasks de um projecto |
| `/sdd-status <proj> <ID>` | Qualquer altura | Detalhe de uma task + estado Linear |
| `/sdd-validate <proj> <spec> [bloco]` | Specs AI-generated | Validar blocos de specs gerais (00-06_*.md) |

### Exemplos

```
/sdd-request dentherm D1-paredes-exteriores     Criar request para paredes exteriores
/sdd-audit dentherm B2                          Verificar se B2 ja esta implementado
/sdd-specify dentherm B2                        Criar spec tecnica para B2
/sdd-plan dentherm B2                           Criar plano + issues Linear para B2
/sdd-close dentherm A2                        Fechar A2 (todos issues Done)
/sdd-archive dentherm A2                      User testa + arquivar A2
/sdd-status dentherm                            Ver todas as tasks do DenTherm
/sdd-validate dentherm 02_ModeloDados           Listar blocos da spec de modelo de dados
```

---

## Comandos — Implementacao (repos de codigo)

| Comando | Quando | O que faz |
|---------|--------|-----------|
| `/sdd-implement <issue-ID>` | Issue atribuido | Implementa seguindo plan.md (Domain -> App -> Infra -> Presentation) |
| `/sdd-check <issue-ID>` | Apos implementar | Verifica contra spec + anti-patterns. Produz relatorio |
| `/end-issue <issue-ID>` | Apos check OK | Commit + PR + merge + fecha issue no Linear |
| `/fix-bug <issue-ID>` | Bug reportado | Investiga + planeia fix + implementa |
| `/fix-bug <descricao>` | Bug novo | Investiga + cria issue + implementa |

### Exemplos

```
/sdd-implement DS-332                       Implementar selector de modulo
/sdd-check DS-332                           Verificar implementacao
/end-issue DS-332                           Fechar issue (commit + PR)
/fix-bug DS-400                             Investigar e corrigir bug
/fix-bug "Ctrl+S nao funciona no projecto novo" Investigar bug sem issue
```

---

## Estrutura de Pastas dos Requests

```
projectos/<projecto>/requests/
├── em-analise/                   Ficheiros simples .md
│   └── C1-dados-gerais.md
├── aprovados/                    Ficheiros .md OU pastas (se ja tem spec)
│   └── B2-identificacao.md
├── em-implementacao/             Pastas completas
│   └── A8-selecionar-modulo/
│       ├── request.md              Pedido original
│       ├── spec.md                 Especificacao tecnica
│       ├── plan.md                 Plano de implementacao
│       └── issues.md               Issues Linear + ordem
├── concluidos/                   Pastas arquivadas
│   └── B1-criar-processo/
│       ├── request.md
│       ├── spec.md
│       └── archive-report.md
└── rejeitados/                   Ficheiros descartados
```

---

## Projectos e Prefixos

| Projecto | Classificacao | Repo codigo | Prefixo Linear |
|----------|---------------|-------------|----------------|
| `denstudio` | [CORE] + [APP] | denstudio | DS-XXX |
| `dentherm` | [THERMAL] | dentherm | DT-XXX |
| `densim` | [SIM] | densim | SM-XXX |

---

## Regras Rapidas

**Planeamento:**
- Request novo -> ficheiro simples em `em-analise/`
- `/sdd-specify` converte ficheiro em pasta (request.md + spec.md)
- `/sdd-plan` adiciona plan.md + issues.md e move para `em-implementacao/`
- `/sdd-close` so quando TODOS os issues Linear estao Done

**Implementacao:**
- Ler ANTI_PATTERNS.md antes de comecar (obrigatorio)
- Modificar existente > Estender > Criar novo
- Ficheiro < 300 LOC, Metodo < 30 LOC
- Interface so com 2+ implementacoes concretas
- Issue >= 13 pontos -> obrigatorio partir

**Cross-module:**
- Dependencias so fluem para baixo: denstudio [CORE]+[APP] -> dentherm [THERMAL] / densim [SIM]
- Se task toca 2+ repos -> partir em sub-tasks
- Implementar denstudio [CORE] primeiro, depois consumidores
- [CORE] e [APP] vivem no mesmo repo (denstudio) — nao e cross-module

---

## Escala de Estimativas (Fibonacci)

### Issues (Linear)

| Pts | Tempo | Notas |
|-----|-------|-------|
| 1 | ~30min | Muito simples |
| 2 | ~1h | Tipico |
| 3 | ~2h | Medio |
| 5 | ~4h | Complexo |
| 8 | ~8h | Grande — avaliar partir |
| 13 | ~16h | **Obrigatorio partir** |

### Planeamento (Requests / Roadmaps) — nao usar no Linear

| Pts | Tempo | Notas |
|-----|-------|-------|
| 34 | ~55h | Feature muito grande — partir em sub-requests |
| 55 | ~90h | Epico — multiplos sub-requests |
| 89 | ~145h | Mega-epico — modulo completo |

---

*Densare AEC SDD — Fevereiro 2026*
