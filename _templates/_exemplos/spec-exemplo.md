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

### RF-01: Campos de Areas e Volume
**Descricao**: Seccao "Areas e Volumes" no painel de detalhe da fracao com campos editaveis.

**Campos**:
| Campo | Propriedade | Tipo | Obrigatorio | Validacao | Unidade |
|-------|-------------|------|-------------|-----------|---------|
| Area util (Ap) | UsefulFloorArea | double | Sim | > 0, <= 10000 | m2 |
| Area bruta (Ag) | GrossFloorArea | double | Nao | >= Ap se preenchido | m2 |
| Pe-direito medio | (calculado ou input) | double | Sim | [1.5, 10.0] | m |
| Volume (V) | Volume | double | Sim | > 0 | m3 |

**Criterios de Aceitacao**:
- [ ] Dado uma fracao seleccionada, quando o painel de detalhe abre, entao a seccao "Areas e Volumes" mostra os 4 campos com valores actuais
- [ ] Dado Ap = 0 (fracao nova), quando o utilizador tenta navegar para calculos, entao recebe aviso de campo obrigatorio
- [ ] Dado Ag preenchido < Ap, quando o utilizador sai do campo, entao mostra erro "Ag deve ser >= Ap"

### RF-02: Calculo Automatico do Volume
**Descricao**: V = Ap x pe-direito. O utilizador pode: (a) introduzir pe-direito e V calcula automaticamente, ou (b) introduzir V directamente e pe-direito calcula como V/Ap.

**Criterios de Aceitacao**:
- [ ] Dado Ap = 100 e pe-direito = 2.7, quando o utilizador confirma, entao V = 270.0 m3
- [ ] Dado Ap = 100 e V = 300 (introduzido manualmente), quando o utilizador confirma, entao pe-direito mostra 3.0 m
- [ ] Dado que o utilizador altera Ap, quando o campo perde foco, entao V recalcula (V = Ap x pe-direito actual)

### RF-03: Persistencia
**Descricao**: Os valores introduzidos sao guardados no SceFraction e persistem com o projecto.

**Criterios de Aceitacao**:
- [ ] Dado que o utilizador preenche Ap e V, quando guarda o projecto e reabre, entao os valores estao preservados
- [ ] Dado que o utilizador altera Ap, quando o valor muda, entao o projecto fica marcado como dirty

### RF-04: Integracao com Motor de Calculo
**Descricao**: Os valores de Ap, Ag e V alimentam GeometricProperties do RehEngine.

**Criterios de Aceitacao**:
- [ ] Dado SceFraction com Ap=100, Ag=120, V=270, quando o motor de calculo arranca, entao GeometricProperties recebe NetFloorArea=100, GrossFloorArea=120, NetHeatedVolume=270

---

## Requisitos Nao-Funcionais

### RNF-01: Performance
- Recalculo de V ao mudar Ap ou pe-direito: instantaneo (< 10ms)
- Nenhuma operacao I/O no recalculo

### RNF-02: Usabilidade
- Campos numericos com spinner (up/down) e input directo
- Unidades visiveis junto ao campo (m2, m, m3)
- Tab order logico: Ap -> pe-direito -> V -> Ag
- Pe-direito com default 2.7 m para fracoes novas

---

## Edge Cases

| # | Cenario | Comportamento Esperado |
|---|---------|------------------------|
| EC-01 | Fracao nova (todos os valores = 0) | Campos vazios, pe-direito pre-preenche 2.7m, V=0 ate Ap ser preenchido |
| EC-02 | Ap alterado para 0 | Erro de validacao, V nao recalcula, pe-direito preservado |
| EC-03 | Pe-direito fora de range (ex: 0.5 ou 15.0) | Erro "Pe-direito deve estar entre 1.5 e 10.0 m" |
| EC-04 | V introduzido manualmente sem Ap | Erro — Ap e obrigatorio |
| EC-05 | Ag < Ap | Erro "Area bruta deve ser >= Area util" |
| EC-06 | Valores muito grandes (Ap = 50000) | Erro acima de 10000 m2 |
| EC-07 | Utilizador altera Ap e depois V manualmente | V manual prevalece; pe-direito recalcula como V/Ap |

---

## Dependencias

### Dependencias Tecnicas
- [x] CommunityToolkit.Mvvm (ja em uso)
- [x] SceFraction entidade (ja existe)
- [x] Dirty tracking (ja implementado — IDirtyTrackable)

### Dependencias de Outras Tasks
- [ ] **B3** (Gestao de Fracoes) — painel de detalhe da fracao. C2 adiciona campos ao painel criado por B3.

### Dependencias Cross-Module

Nenhuma. Tudo vive no `dentherm`:
- Modelo: `DenTherm.Sce.Core.Domain.Entities.SceFraction`
- UI: `DenTherm.UI.ViewModels` / `DenTherm.UI.Views.Panels`

### Classificacao Cross-Module

- [x] Esta task toca APENAS 1 repositorio? -> Sim (`dentherm`)
- [ ] Esta task toca 2+ repositorios? -> Nao
- [ ] Esta task depende de algo que NAO EXISTE noutro modulo? -> Nao

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

- [x] ~~Implementar antes ou depois de B3?~~ Independente — carrega fracao unica do projecto.

---

## Historico

| Data | Autor | Alteracao |
|------|-------|-----------|
| 2026-02-10 | Claude (SDD) | Criacao — reclassificacao [CORE]->[THERMAL], auditoria de codigo, SPECIFIED |

<!--
EXEMPLO: Esta e uma spec real da task C2-areas-volumes do DenTherm.
Pontos chave a notar:
1. Auditoria do codigo existente (o que JA existe vs o que falta)
2. Reclassificacao justificada (CORE -> THERMAL)
3. Criterios de aceitacao em formato Given/When/Then
4. Edge cases concretos com comportamento esperado
5. Fora de scope explicito
6. Quality gate preenchido com decisoes de complexidade
-->
