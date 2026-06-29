# Pedido: Areas e Volumes da fracao

> **Estado**: em-analise · **Pedido por**: Nuno Nunes · **Owner funcional**: Nuno Nunes · **Data**: 2026-02-10
> **Dominio**: AEC · **Area**: THERMAL · **Camada**: UI · **Regulado**: nao
> **Capacidade-mae**: `FUNCIONALIDADES.md` -> `CAP-DT-C-002` ("Deve permitir introduzir area util, pe-direito e volume da fracao")

## 1. Job Story
Quando estou a preencher os dados de uma fracao, quero introduzir a area util, o pe-direito e o volume, para que o motor de calculo termico (Nic, Nvc, fator de forma, ganhos) possa arrancar.

## 2. Porque agora
- Ap (area util) e V (volume) sao usados em TODOS os calculos REH. Sem eles nenhum calculo corre. E o segundo passo funcional depois da identificacao do processo.

## 3. Criterios de Aceitacao
<!-- 1 CA = 1 obrigacao. Erros e edge cases = CA proprios. [bloqueia-close] nos que travam o fecho. -->

### CA-01 — Seccao mostra os campos com os valores atuais  [bloqueia-close]
- **Dado** uma fracao com Ap=100, Ag=120, V=270 ja guardados
- **Quando** abro o painel de detalhe da fracao
- **Entao** a seccao "Areas e Volumes" mostra os 4 campos (Ap, Ag, pe-direito, V) preenchidos com esses valores e as unidades (m2, m, m3)
- **Prova**: smoke manual · projeto-fixture com a fracao acima · observar o painel aberto

### CA-02 — Volume calcula a partir de area x pe-direito  [bloqueia-close]
- **Dado** Ap=100 introduzido
- **Quando** introduzo pe-direito=2,7
- **Entao** V mostra 270,0 m3 (separador decimal PT = virgula)
- **Prova**: teste auto do ViewModel · caso (Ap=100, pe=2,7) -> V=270,0

### CA-03 — Pe-direito calcula a partir de volume manual  [bloqueia-close]
- **Dado** Ap=100 introduzido
- **Quando** introduzo V=300 diretamente
- **Entao** pe-direito mostra 3,0 m e o valor manual de V prevalece
- **Prova**: teste auto do ViewModel · caso (Ap=100, V=300) -> pe=3,0

### CA-04 — Area bruta menor que a util e recusada  [bloqueia-close]
- **Dado** Ap=100 introduzido
- **Quando** introduzo Ag=80 e saio do campo
- **Entao** mostra o erro "Area bruta deve ser >= Area util" e o valor nao e aceite
- **Prova**: teste auto de validacao · caso (Ap=100, Ag=80)

### CA-05 — Dados sobrevivem a guardar/reabrir  [bloqueia-close]
- **Dado** uma fracao onde preenchi Ap=100 e V=270
- **Quando** guardo o projeto, fecho e reabro
- **Entao** Ap=100 e V=270 continuam la
- **Prova**: teste auto de round-trip (guardar -> ler) · assercao sobre SceFraction

### CA-06 — Valores alimentam o motor de calculo  [bloqueia-close]
- **Dado** uma fracao com Ap=100, Ag=120, V=270
- **Quando** o motor de calculo arranca
- **Entao** GeometricProperties recebe NetFloorArea=100, GrossFloorArea=120, NetHeatedVolume=270
- **Prova**: teste auto de integracao · assercao sobre GeometricProperties

## 4. Dependencias
| Dependencia | Estado | Owner | Como testar se indisponivel |
|---|---|---|---|
| B3 (painel de detalhe da fracao) | em curso | dentherm | carregar a fracao unica do projeto via LoadFrom() ate B3 existir |

## 5. Fora de ambito (explicito)
- **Excluido**: gestao de espacos/divisoes individuais (SceSpace) -> request futuro · calculo de Ap como soma das divisoes -> request futuro · desenho/planta para determinar areas -> fora do R1.1 · fator de forma (calculado pelo motor, nao editavel)
- **Preparado mas nao implementado**: nada

## 6. Questoes em aberto
- Nenhuma (independente de B3 — carrega a fracao unica do projeto).

## 7. Notas (so contexto NAO-normativo)
- Pe-direito por defeito 2,7 m para fracoes novas (valor tipico em Portugal).
- O dominio ja existe: `SceFraction` tem UsefulFloorArea/GrossFloorArea/Volume + GetAverageHeight() + IsValid(). Esta task e UI + binding.

<!--
EXEMPLO (request real C2-areas-volumes do DenTherm, reescrito no formato de Criterios de Aceitacao).
Pontos a notar:
1. Job Story (intencao), nao prosa "como imagino que funcione".
2. Cada CA = UMA obrigacao, observavel e mensuravel, com uma linha Prova (o que demonstra que passou).
3. NF/i18n que importa (virgula decimal PT) entra no "Entao", nao numa nota.
4. Dependencias em seccao propria, com "como testar se indisponivel".
5. Fora de ambito explicito. Notas = so contexto que NAO bloqueia o fecho.
6. Cada CA vira um RF na spec (RF <-> CA bidirecional) e a linha Prova vira a verificacao do /sdd-check.
Iniciar a analise: /specify dentherm C2
-->
