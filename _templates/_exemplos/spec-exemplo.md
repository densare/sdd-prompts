# Spec: C2 — Areas e Volumes da Fracao

> **Estado**: SPECIFIED
> **Classificacao**: [THERMAL]
> **Repositorio**: dentherm
> **Criado**: 2026-02-10

---

## Resumo

Seccao de dados geometricos da fracao no painel de detalhe termico: area util (Ap), area bruta (Ag), pe-direito medio, e volume interior com calculo automatico V = Ap x pe-direito.

---

## Contexto

### Problema
Ap (area util de pavimento) e V (volume interior) sao parametros fundamentais usados em TODOS os calculos termicos REH — Nic, Nvc, Ntc, Qa, fator de forma, ganhos internos. Sem estes valores preenchidos, nenhum calculo pode correr. Atualmente, `SceFraction` tem as propriedades no modelo de dominio mas nao existe UI para o utilizador as preencher.

### Utilizadores Afetados
Peritos qualificados (PQ) que preenchem dados do edificio para certificacao REH.

### Valor de Negocio
E o segundo passo funcional apos a identificacao do processo. Sem Ap e V, o motor de calculo nao pode arrancar. Bloqueia todos os calculos.

### Reclassificacao: [CORE] → [THERMAL]

O request original indicava [CORE], mas:
- `SceFraction` (com UsefulFloorArea, GrossFloorArea, Volume) vive em `DenTherm.Sce.Core.Domain.Entities`
- A UI sera um painel do DenTherm, nao do DenStudio generico
- **Pergunta**: "Se outro modulo precisasse de areas, faria sentido estar no DenStudio?" — Nao, cada modulo teria o seu conceito de area

> Classificacao correcta: **[THERMAL]**, repositorio `dentherm`.

---

## Codigo Existente (Auditoria)

### Modelo de Dominio — JA EXISTE

| Classe | Propriedade | Descricao | Estado |
|--------|-------------|-----------|--------|
| `SceFraction` | `UsefulFloorArea` | Area util Ap (m2) | Propriedade existe |
| `SceFraction` | `GrossFloorArea` | Area bruta Ag (m2) | Propriedade existe |
| `SceFraction` | `Volume` | Volume interior V (m3) | Propriedade existe |
| `SceFraction` | `GetAverageHeight()` | Pe-direito medio = V / Ap | Metodo existe |
| `SceFraction` | `IsValid()` | Valida Ap > 0, V > 0, Ap <= Ag | Validacao existe |

### UI — NAO EXISTE

| Componente | Estado |
|-----------|--------|
| ViewModel para editar areas/volumes da fracao | NAO EXISTE |
| View/Panel AXAML para campos Ap, Ag, pe-direito, V | NAO EXISTE |

---

## Requisitos Funcionais

> **RF ↔ CA bidirecional (obrigatorio)**: cada RF referencia o(s) CA do request que satisfaz (`Satisfaz: CA-NN`), e cada CA do request mapeia para um RF. **Nenhum RF sem CA** (= requisito nao-testavel) e **nenhum CA orfao**. Cada criterio carrega a linha **Prova** do request — o mecanismo que o demonstra; e isso que `/sdd-check` executa e `/sdd-close` lista. **1 CA = 1 obrigacao.**
>
> Mapa deste spec: CA-01→RF-01 · CA-02,CA-03→RF-02 · CA-04→RF-03 · CA-05→RF-04 · CA-06→RF-05.

### RF-01: Campos de Areas e Volume
**Descricao**: Seccao "Areas e Volumes" no painel de detalhe da fracao com campos editaveis (Ap, Ag, pe-direito, V), unidades visiveis, valores atuais carregados.
**Satisfaz**: CA-01

**Campos**:
| Campo | Propriedade | Tipo | Obrigatorio | Validacao | Unidade |
|-------|-------------|------|-------------|-----------|---------|
| Area util (Ap) | UsefulFloorArea | double | Sim | > 0, <= 10000 | m2 |
| Area bruta (Ag) | GrossFloorArea | double | Nao | >= Ap se preenchido | m2 |
| Pe-direito medio | (calculado ou input) | double | Sim | [1.5, 10.0] | m |
| Volume (V) | Volume | double | Sim | > 0 | m3 |

**Criterios de Aceitacao**:
- [ ] **CA-01** — Dado uma fracao com Ap=100, Ag=120, V=270, quando o painel de detalhe abre, entao a seccao "Areas e Volumes" mostra os 4 campos com esses valores e as unidades (m2, m, m3)
  **Prova**: smoke manual · projeto-fixture com a fracao acima · observar o painel aberto

### RF-02: Calculo Automatico do Volume / Pe-direito
**Descricao**: V = Ap x pe-direito. O utilizador pode (a) introduzir pe-direito e V calcula automaticamente, ou (b) introduzir V diretamente e pe-direito calcula como V/Ap. Separador decimal PT (virgula).
**Satisfaz**: CA-02, CA-03

**Criterios de Aceitacao**:
- [ ] **CA-02** — Dado Ap=100, quando se introduz pe-direito=2,7, entao V mostra 270,0 m3
  **Prova**: teste auto do ViewModel · caso (Ap=100, pe=2,7) -> V=270,0
- [ ] **CA-03** — Dado Ap=100, quando se introduz V=300 manualmente, entao pe-direito mostra 3,0 m e o V manual prevalece
  **Prova**: teste auto do ViewModel · caso (Ap=100, V=300) -> pe=3,0

### RF-03: Validacao de Area Bruta
**Descricao**: Ag, se preenchido, tem de ser >= Ap; caso contrario erro e o valor nao e aceite.
**Satisfaz**: CA-04

**Criterios de Aceitacao**:
- [ ] **CA-04** — Dado Ap=100, quando se introduz Ag=80 e se sai do campo, entao mostra "Area bruta deve ser >= Area util" e o valor nao e aceite
  **Prova**: teste auto de validacao · caso (Ap=100, Ag=80)

### RF-04: Persistencia
**Descricao**: Os valores introduzidos sao guardados no SceFraction e persistem com o projeto.
**Satisfaz**: CA-05

**Criterios de Aceitacao**:
- [ ] **CA-05** — Dado Ap=100 e V=270 preenchidos, quando se guarda o projeto e se reabre, entao os valores estao preservados
  **Prova**: teste auto de round-trip (guardar -> ler) · assercao sobre SceFraction

### RF-05: Integracao com Motor de Calculo
**Descricao**: Os valores de Ap, Ag e V alimentam GeometricProperties do RehEngine.
**Satisfaz**: CA-06

**Criterios de Aceitacao**:
- [ ] **CA-06** — Dado SceFraction com Ap=100, Ag=120, V=270, quando o motor de calculo arranca, entao GeometricProperties recebe NetFloorArea=100, GrossFloorArea=120, NetHeatedVolume=270
  **Prova**: teste auto de integracao · assercao sobre GeometricProperties

> **Calculo regulado** (resultado com valor legal): incluir um **CA de nao-regressao** — "toda a suite de oraculos mantem-se dentro da tolerancia aprovada; qualquer diff exige aprovacao do dono" (Prova = runner de regressao/legacy em CI, label `validate-against-legacy`). Oraculos por ID/versao estaveis; **tolerancia zero** no veredicto que cruza um limite legal; comparar tambem **parcelas intermedias**, nao so o total. *(Nao se aplica a esta task — C2 e input de UI, nao calculo; serve de lembrete para specs THERMAL de calculo.)*

---

## Requisitos Nao-Funcionais

### RNF-01: Performance
- Recalculo de V ao mudar Ap ou pe-direito: instantaneo (< 10ms), sem I/O. *(Nao bloqueia close — nao e CA; se viesse a bloquear, viraria CA com Prova = benchmark.)*

### RNF-02: Usabilidade
- Campos numericos com spinner e input directo; unidades visiveis; tab order Ap -> pe -> V -> Ag; pe-direito default 2,7 m em fracoes novas.

---

## Edge Cases

| # | Cenario | Comportamento Esperado | CA |
|---|---------|------------------------|----|
| EC-01 | Fracao nova (valores = 0) | Campos vazios, pe-direito pre-preenche 2,7m, V=0 ate Ap ser preenchido | (RF-01) |
| EC-02 | Ap alterado para 0 | Erro de validacao, V nao recalcula, pe-direito preservado | (RF-02) |
| EC-03 | Pe-direito fora de range (0.5 ou 15.0) | Erro "Pe-direito deve estar entre 1.5 e 10.0 m" | (RF-02) |
| EC-04 | V manual sem Ap | Erro — Ap e obrigatorio | (RF-02) |
| EC-05 | Ag < Ap | Erro "Area bruta deve ser >= Area util" | CA-04 |
| EC-06 | Valores muito grandes (Ap = 50000) | Erro acima de 10000 m2 | (RF-01) |
| EC-07 | Ap e depois V manual | V manual prevalece; pe-direito recalcula como V/Ap | CA-03 |

---

## Dependencias

### Dependencias Tecnicas
- [x] CommunityToolkit.Mvvm (ja em uso)
- [x] SceFraction entidade (ja existe)
- [x] Dirty tracking (ja implementado — IDirtyTrackable)

### Dependencias de Outras Tasks
- [ ] **B3** (Gestao de Fracoes) — painel de detalhe da fracao. C2 adiciona campos ao painel criado por B3. **Nao bloqueante**: ate B3, carrega a fracao unica do projeto via LoadFrom() (ver request §4).

### Dependencias Cross-Module
Nenhuma. Tudo vive no `dentherm` (modelo `DenTherm.Sce.Core.Domain.Entities.SceFraction`; UI `DenTherm.UI`).

### Classificacao Cross-Module
- [x] Toca APENAS 1 repositorio (`dentherm`). Nao depende de algo inexistente noutro modulo.

---

## Fora de Scope

**NAO sera implementado nesta task**:
- Gestao de espacos/divisoes individuais (SceSpace) — request futuro
- Calculo automatico de Ap como soma de areas das divisoes — request futuro
- Desenho/planta para determinar areas — fora do scope R1.1
- Fator de forma (FF = Aenv/V) — calculado pelo motor, nao editavel

---

## Quality Gate: SPECIFY

### Verificacao de Scope
- [x] Task faz UMA coisa — input de areas e volume da fracao
- [x] Requisitos independentes de implementacao
- [x] Nao generaliza prematuramente (sem espacos/divisoes)
- [x] Nao mistura UI + logica dominio — dominio ja existe, task e puramente UI + binding

### Verificacao RF ↔ CA (bidirecional)
- [x] Todo o RF referencia >=1 CA do request (`Satisfaz:`) e todo o CA do request mapeia para um RF (CA-01..06 cobertos)
- [x] Todo o CA tem linha **Prova** executavel (sem Prova = CA vago -> BLOCK)
- [x] NF que bloqueiam expressos como CA, nao em prosa (aqui nenhum NF bloqueia; RNF-01/02 sao desejaveis, declarados como tal)

### Classes/Services Existentes Relacionados

| Classe/Service | Localizacao | Relacao | Reutilizar? |
|----------------|-------------|---------|-------------|
| SceFraction | DenTherm.Sce.Core.Domain.Entities | Modelo com Ap, Ag, V | Sim — binding directo |
| SceFraction.GetAverageHeight() | Mesmo | Calcula pe-direito = V/Ap | Sim — para display |
| SceFraction.IsValid() | Mesmo | Valida Ap>0, V>0, Ap<=Ag | Sim — para validacao |
| GeometricProperties | DenTherm.Sce.Core.Calculation | DTO para motor calculo | Referencia — nao modificar |
| ProcessIdentificationViewModel | DenTherm.UI.ViewModels | Padrao de ViewModel existente | Referencia — seguir padrao |

### Decisao de Complexidade
- Novo namespace? **Nao** — ViewModel vai para namespace existente
- Nova interface? **Nao** — sem abstraccoes
- Risco de over-engineering? **Baixo** — 4 campos, 1 formula, binding a entidade existente

---

## Questoes em Aberto

- [x] ~~Implementar antes ou depois de B3?~~ Independente — carrega fracao unica do projeto.

---

## Historico

| Data | Autor | Alteracao |
|------|-------|-----------|
| 2026-02-10 | Claude (SDD) | Criacao — reclassificacao [CORE]->[THERMAL], auditoria de codigo, RF<->CA, SPECIFIED |

<!--
EXEMPLO: spec real da task C2-areas-volumes do DenTherm, no formato RF <-> CA.
Pontos chave a notar:
1. Auditoria do codigo existente (o que JA existe vs o que falta)
2. Reclassificacao justificada (CORE -> THERMAL)
3. RF <-> CA bidirecional: cada RF diz que CA satisfaz; cada CA do request tem um RF. Sem RF orfao, sem CA orfao.
4. Cada CA carrega a linha Prova do request (teste auto / oraculo / benchmark / smoke) — e o que o /sdd-check executa.
5. Edge cases concretos; NF que NAO bloqueiam ficam como RNF (se bloqueassem, virariam CA).
6. Lembrete de calculo regulado (nao-regressao + tolerancia zero) para specs THERMAL de calculo.
-->
