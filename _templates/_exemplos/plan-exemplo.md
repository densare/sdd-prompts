# Plan: C2 — Areas e Volumes da Fracao

> **Estado**: PLANNED
> **Spec**: [spec.md](spec.md)
> **Classificacao**: [THERMAL]
> **Repositorio**: dentherm
> **Criado**: 2026-02-10

---

## Auditoria (Passo 0)

| Requisito | CA | Estado | Evidencia |
|-----------|----|--------|-----------|
| RF-01: Campos Ap, Ag, pe-direito, V | CA-01 | NAO INICIADO | SceFraction tem propriedades, mas nenhum ViewModel/View existe |
| RF-02: Calculo automatico V <-> pe | CA-02, CA-03 | NAO INICIADO | SceFraction.GetAverageHeight() existe (inverso), mas sem UI |
| RF-03: Validacao Ag >= Ap | CA-04 | NAO INICIADO | SceFraction.IsValid() existe, mas sem feedback de UI |
| RF-04: Persistencia | CA-05 | PARCIAL | SceFraction persiste via projecto, mas sem binding de UI |
| RF-05: Integracao motor calculo | CA-06 | DONE | GeometricProperties ja le de SceFraction |

**Resultado**: NAO INICIADO (RF-01/02/03 sao o core da task). RF-05 nao precisa de trabalho.

---

## Resumo do Plano

Criar um ViewModel e View para editar areas e volumes da fracao, seguindo exactamente o padrao de `BuildingDataPanelViewModel` + `BuildingDataPanel.axaml`. O ViewModel faz binding directo a `SceFraction` via `LoadFrom()/WriteTo()`, com logica de recalculo bidirecional (V = Ap x pe <-> pe = V / Ap). Registar o painel no `DenThermUIProvider`.

---

## Analise da Spec

### Requisitos Mapeados

| Requisito | Complexidade | Prioridade |
|-----------|--------------|------------|
| RF-01: Campos de areas e volume | Baixa | P1 |
| RF-02: Calculo automatico volume | Media | P1 |
| RF-03: Persistencia | Baixa | P1 |
| RF-04: Integracao motor calculo | Nenhuma (ja feito) | - |

### Riscos Identificados

| Risco | Probabilidade | Impacto | Mitigacao |
|-------|---------------|---------|-----------|
| Sem seleccao de fracao (B3 nao feito) | Alta | Medio | Carregar fracao unica do projecto; LoadFrom() reutilizavel |
| Ciclo de recalculo (Ap->V->pe->V...) | Media | Baixo | Flag `_isRecalculating` para bloquear ciclos |

---

## Analise do Codigo Existente

### Pesquisa Realizada

| O que procurei | Onde procurei | O que encontrei |
|----------------|---------------|-----------------|
| ViewModel para fracao/areas | DenTherm.UI/ViewModels/ | NAO EXISTE — nenhum FractionViewModel |
| View AXAML para areas | DenTherm.UI/Views/Panels/ | NAO EXISTE — nenhum FractionDataPanel |
| Padrao de ViewModel | ProcessIdentificationViewModel, BuildingDataPanelViewModel | ViewModelBase + SetProperty + LoadFrom/WriteTo |
| Padrao de panel AXAML | BuildingDataPanel.axaml | Grid layout, NumericUpDown, TextBlock labels |
| Registo de paineis | DenThermUIProvider.cs | GetPanels() + CreateView() + RegisterServices() |
| Propriedades de area | SceFraction.cs | UsefulFloorArea, GrossFloorArea, Volume, GetAverageHeight() |

### Codigo Existente que Pode Ser Reutilizado/Modificado

| Ficheiro Existente | O que ja faz | O que precisa de mudar | Decisao |
|--------------------|-------------|------------------------|---------|
| `SceFraction.cs` | UsefulFloorArea, GrossFloorArea, Volume, GetAverageHeight(), IsValid() | Nada | Reutilizar (binding directo) |
| `ViewModelBase.cs` | INotifyPropertyChanged + SetProperty | Nada | Reutilizar (herdar) |
| `BuildingDataPanelViewModel.cs` | Padrao de data entry (158 LOC) | Nada — e referencia | Seguir padrao |
| `DenThermUIProvider.cs` | Registo de paineis (347 LOC) | +1 panel, +1 case, +1 DI | Modificar |

---

## Arquitectura

### Componentes Envolvidos

```
SceFraction (Domain - JA EXISTE)
      ^ LoadFrom/WriteTo
FractionDataPanelViewModel (Presentation - NOVO)
      ^ DataBinding
FractionDataPanel.axaml (Presentation - NOVO)
      ^ Registado em
DenThermUIProvider (Presentation - MODIFICAR)
```

### Decisoes de Design

1. **Seguir padrao BuildingDataPanelViewModel**: ViewModel simples com ViewModelBase, LoadFrom/WriteTo, sem servicos injectados. Justificacao: consistencia (AP-08).
2. **Recalculo bidirecional com flag anti-ciclo**: Flag `_isRecalculating` evita ciclos infinitos entre campos interdependentes.
3. **Sem interface para ViewModel**: Classe directa, so 1 implementacao prevista (AP-01).
4. **Independente de B3**: LoadFrom() aceita qualquer SceFraction. Hoje carrega fracao unica; quando B3 existir, carrega a seleccionada.

---

## Ficheiros a Alterar

### Ficheiros Existentes a Modificar (PRIMEIRO)

| Ficheiro | O que Existe | O que Vai Mudar | Camada |
|----------|-------------|-----------------|--------|
| `src/DenTherm.UI/Providers/DenThermUIProvider.cs` | Registo de paineis (347 LOC) | +1 panel em GetPanels(), +1 case em CreateView(), +1 registo DI (~+15 LOC) | Presentation |

### Ficheiros Novos (so se justificado)

| Ficheiro | Camada | Proposito | Porque nao modificar existente? |
|----------|--------|-----------|--------------------------------|
| `ViewModels/FractionDataPanelViewModel.cs` | Presentation | ViewModel com 4 props + LoadFrom/WriteTo + recalculo | Nao existe ViewModel para fracao |
| `Views/Panels/FractionDataPanel.axaml` | Presentation | View com 4 NumericUpDown + labels + unidades | Nao existe View para dados de fracao |
| `Views/Panels/FractionDataPanel.axaml.cs` | Presentation | Code-behind (InitializeComponent) | Companion obrigatorio do AXAML |

**Racio**: 1 modificado, 3 novos. Justificacao: nao existe nenhuma UI de fracao. Domain ja existe (0 novos).

---

## Passos de Implementacao

### Passo 1: FractionDataPanelViewModel
**Objetivo**: ViewModel com propriedades Ap, Ag, pe-direito, V e logica de recalculo.

**Acoes**:
1. [ ] Criar `FractionDataPanelViewModel : ViewModelBase` (~100 LOC)
2. [ ] Propriedades: UsefulFloorArea (double), GrossFloorArea (double?), AverageCeilingHeight (double, default 2.7), Volume (double)
3. [ ] Recalculo: Ap ou pe muda -> V = Ap x pe. V muda -> pe = V / Ap. Flag anti-ciclo.
4. [ ] Validacao: Ap > 0, pe em [1.5, 10.0], Ag >= Ap, V > 0
5. [ ] LoadFrom(SceFraction): ler valores do dominio
6. [ ] WriteTo(SceFraction): escrever valores de volta

**Ficheiros**:
- `src/DenTherm.UI/ViewModels/FractionDataPanelViewModel.cs`

**Validacao**:
- [ ] LOC < 120
- [ ] Sem interfaces novas
- [ ] LoadFrom/WriteTo seguem padrao existente

---

### Passo 2: FractionDataPanel.axaml
**Objetivo**: View com 4 campos numericos e labels.

**Acoes**:
1. [ ] Criar UserControl `FractionDataPanel` (~80 LOC AXAML)
2. [ ] Layout: Grid 4 rows x 3 columns (label | NumericUpDown | unidade)
3. [ ] NumericUpDown com ranges corretos
4. [ ] Tab order: Ap -> pe -> V -> Ag

**Ficheiros**:
- `src/DenTherm.UI/Views/Panels/FractionDataPanel.axaml`
- `src/DenTherm.UI/Views/Panels/FractionDataPanel.axaml.cs`

**Validacao**:
- [ ] AXAML < 100 LOC
- [ ] Unidades visiveis (m2, m, m3)

---

### Passo 3: Registo no DenThermUIProvider
**Objetivo**: Integrar o painel no sistema de UI do DenTherm.

**Acoes**:
1. [ ] Adicionar `services.AddTransient<FractionDataPanelViewModel>()` em RegisterServices()
2. [ ] Adicionar PanelDefinition em GetPanels()
3. [ ] Adicionar case em CreateView()

**Ficheiros**:
- `src/DenTherm.UI/Providers/DenThermUIProvider.cs` (MODIFICAR)

**Validacao**:
- [ ] Ficheiro total < 370 LOC
- [ ] Padrao consistente com paineis existentes

---

### Passo 4: Testes (provas dos CA)
**Objetivo**: Materializar a linha `Prova:` de cada CA. As provas automaticas correm em CI; a prova de smoke (CA-01) e apresentada ao operador no `/sdd-check`.

**Acoes** (cada uma e a Prova de um CA da spec):
1. [ ] CA-02: Ap=100, pe=2,7 -> V=270,0
2. [ ] CA-03: Ap=100, V=300 (manual) -> pe=3,0
3. [ ] CA-04: Ag=80 com Ap=100 -> erro, valor recusado
4. [ ] CA-05: round-trip guardar->ler preserva Ap=100, V=270
5. [ ] CA-06: GeometricProperties recebe Net=100 / Gross=120 / Volume=270
6. [ ] (EC) Alterar Ap com pe fixo -> V recalcula · Ap=0 -> V nao recalcula · pe fora de range -> erro
7. [ ] (suporte) LoadFrom carrega · WriteTo escreve
8. [ ] CA-01 = **smoke manual** (ver "Smoke manual" abaixo) — nao tem prova automatica; verifica-se a correr a app

**Ficheiros**:
- `tests/DenTherm.UI.Tests/ViewModels/FractionDataPanelViewModelTests.cs`
- `tests/DenTherm.UI.Tests/FractionPersistenceTests.cs` (CA-05)
- `tests/DenTherm.Sce.Core.Tests/FractionEngineIntegrationTests.cs` (CA-06)

**Validacao**:
- [ ] Toda a linha `Prova:` automatica da spec tem teste correspondente (CA-02..06)
- [ ] CA-01 (smoke) listado na seccao "Smoke manual"
- [ ] Testes verificam OUTPUT, nao estado interno

---

## Testes

> **Os testes derivam da linha `Prova:` de cada CA da spec.** Cada CA `[bloqueia-close]` tem aqui a sua prova planeada (auto ou smoke). E esta lista que o `/sdd-check` corre e o `/sdd-close` exige.

### Testes Unitarios / Integracao (xUnit + FluentAssertions) — provas automaticas

| Teste | CA | Descricao | Ficheiro |
|-------|----|-----------|----------|
| `RecalculateVolume_WhenApAndCeilingHeightSet` | CA-02 | (Ap=100, pe=2,7) -> V=270,0 | FractionDataPanelViewModelTests.cs |
| `RecalculateCeilingHeight_WhenVolumeSetManually` | CA-03 | (Ap=100, V=300) -> pe=3,0 | Mesmo |
| `Validation_GrossAreaLessThanUseful` | CA-04 | (Ap=100, Ag=80) -> erro, valor recusado | Mesmo |
| `RoundTrip_PersistsAreasAndVolume` | CA-05 | guardar -> ler preserva Ap=100, V=270 | FractionPersistenceTests.cs |
| `GeometricProperties_FedFromFraction` | CA-06 | Ap/Ag/V -> Net/Gross/Volume no motor | FractionEngineIntegrationTests.cs |
| `RecalculateVolume_WhenApChanges` | (RF-02 EC) | V ajusta com pe fixo | FractionDataPanelViewModelTests.cs |
| `NoRecalculation_WhenApIsZero` | (EC-02) | V preservado | Mesmo |
| `Validation_CeilingHeightOutOfRange` | (EC-03) | Erro pe fora [1.5, 10.0] | Mesmo |
| `LoadFrom_PopulatesViewModelFromFraction` | (suporte CA-01) | Binding domain -> VM | Mesmo |
| `WriteTo_UpdatesFractionFromViewModel` | (suporte CA-05) | Binding VM -> domain | Mesmo |

### Smoke manual (da linha `Prova:` dos CA) — comportamento observavel

> So os CA cuja Prova e "smoke manual" (comportamento que so se ve a correr a app). O `/sdd-check` apresenta estes ao operador.

| # | CA | Receita | PASS = |
|---|----|---------|--------|
| 1 | CA-01 | Abrir projeto-fixture com fracao (Ap=100, Ag=120, V=270) -> abrir painel de detalhe da fracao | Seccao "Areas e Volumes" mostra os 4 campos com 100 / 120 / 2,7 / 270 e as unidades (m2, m, m3) |

---

## Dependencias de Ordem

```
Passo 1 (ViewModel) --> Passo 2 (View — precisa do ViewModel)
                    --> Passo 4 (Testes — testam o ViewModel)
Passo 1 + 2 -------> Passo 3 (Registo — precisa de ambos)
```

---

## Estimativa

| Passo | Pontos |
|-------|--------|
| Passo 1: FractionDataPanelViewModel | 1 |
| Passo 2: FractionDataPanel.axaml | 1 |
| Passo 3: Registo DenThermUIProvider | 0.5 |
| Passo 4: Testes | 1 |
| **Total** | **3** (~2h) |

---

## Quality Gate: PLAN

### Verificacao de Camadas C# (Clean Architecture)

| Ficheiro Novo | Camada Proposta | Correta? | Notas |
|---------------|-----------------|----------|-------|
| FractionDataPanelViewModel.cs | Presentation | Sim | ViewModel e camada de apresentacao |
| FractionDataPanel.axaml | Presentation | Sim | View AXAML |
| FractionDataPanelViewModelTests.cs | Testes | Sim | Unit test |

### Verificacao Search Before Create
- [x] Pesquisa de codigo existente feita (seccao preenchida)
- [x] Cada ficheiro novo justificado
- [ ] Mais modificados do que novos? **Nao** — Justificacao: nao existe nenhuma UI de fracao

### Padrao Escolhido

| Aspeto | Padrao Existente | Padrao Escolhido | Justificacao |
|--------|------------------|------------------|--------------|
| ViewModel | ViewModelBase + SetProperty | Mesmo | AP-08: consistencia |
| Load/Save | LoadFrom/WriteTo | Mesmo | AP-08: consistencia |
| View | UserControl + Grid + NumericUpDown | Mesmo | AP-08: consistencia |
| Registo | DenThermUIProvider.GetPanels() | Mesmo | AP-08: consistencia |

### Verificacao de Interfaces
Nenhuma interface nova criada. (AP-01 OK)

### Anti-Patterns Verificados
- [x] AP-01: Zero interfaces novas
- [x] AP-02: Zero seguranca (dados nao-sensiveis)
- [x] AP-03: N/A (sem atributos de seguranca)
- [x] AP-04: Reutiliza SceFraction, ViewModelBase, padrao existente
- [x] AP-05: Todos os ficheiros < 120 LOC
- [x] AP-06: Tudo na camada Presentation (correcto)
- [x] AP-07: Zero stubs
- [x] AP-08: Segue padroes existentes

---

## Dependencias Cross-Module

### Pre-condicoes

| Dependencia | Modulo | Estado Actual | Bloqueante? |
|-------------|--------|---------------|-------------|
| SceFraction entidade | [THERMAL] | IMPLEMENTED | Nao |
| ViewModelBase | [THERMAL] | IMPLEMENTED | Nao |
| B3 (seleccao fracao) | [CORE]+[THERMAL] | PLANNED | **Nao** — carrega fracao unica |

---

## Checklist Pre-Implementacao

- [x] Spec aprovada (SPECIFIED)
- [x] Dependencias tecnicas disponiveis
- [x] Dependencias cross-module: nenhuma bloqueante
- [x] Quality Gates verificados
- [x] Estimativa: 3 pontos (~2h)

---

## Notas de Implementacao

- **Fracao unica**: Ate B3, ha 1 fracao por projecto. ViewModel carrega essa automaticamente.
- **Pe-direito nao e propriedade de SceFraction**: SceFraction guarda Volume. Pe-direito e derivado ou input para calcular V. O ViewModel mantém estado local sincronizado via LoadFrom/WriteTo.
- **Anti-ciclo**: Flag `_isRecalculating` nos setters para evitar recalculo recursivo.

---

## Historico

| Data | Autor | Alteracao |
|------|-------|-----------|
| 2026-02-10 | Claude (SDD) | Criacao — auditoria, plano, PLANNED |

<!--
EXEMPLO: Este e um plan real da task C2-areas-volumes do DenTherm.
Pontos chave a notar:
1. Auditoria (passo 0) verifica o que JA existe antes de planear, com a coluna CA (RF<->CA)
2. Analise do codigo existente com tabelas de pesquisa e decisoes
3. Justificacao para cada ficheiro novo
4. Verificacao de todos os 8 anti-patterns
5. Estimativa em Fibonacci com breakdown por passo
6. Notas de implementacao com gotchas concretas
7. Os testes derivam da linha Prova: de cada CA da spec; o que e smoke manual (CA-01)
   fica na seccao "Smoke manual" — e o que o /sdd-check apresenta e o /sdd-close exige.
-->
