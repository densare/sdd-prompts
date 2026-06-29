# Pedido: [Nome curto e concreto da funcionalidade]

> **Estado**: rascunho | em-analise | aprovado | em-implementacao | superseded | cancelado
> **Pedido por**: [NOME] · **Owner funcional**: [NOME] · **Data**: [AAAA-MM-DD]
> **Dominio**: AEC | Cloud   ·   **Area**: THERMAL | SIM | CORE | PLATFORM | APP:<produto> | ADMIN | SDK
> **Camada**: Dominio | API | UI | SDK   ·   **Regulado**: sim | nao
> **Capacidade-mae**: `FUNCIONALIDADES.md` → `CAP-<APP>-<EPIC>-<NNN>` ("<linha 'Deve fazer Y'>")

<!-- `Regulado: sim` (calculo com valor legal) obriga ao addendum de nao-regressao nos CA (ver seccao 3). -->
<!-- Prioridade vive no Linear, nao no request (nao duplicar estado). -->
<!-- Escalonamento (nao encher de cerimonia): Nucleo = 1 capacidade, 1-3 CA, sem deps/NF/regulacao. Completo = feature/regulado/toca deps ou NF. Bug = variante no fim deste ficheiro. -->
<!-- Variante de cabecalho generica (fora-Densare): Estado · Pedido por · Owner · Data · Capacidade-mae. Os campos Dominio/Area/Camada/Regulado sao a taxonomia Densare. -->


---

## 1. Objetivo / Job Story

<!-- Produto/user-facing → Job Story. Trabalho tecnico (refactor/migracao/infra) → frase de Objetivo direta. NAO inventar utilizador onde nao ha. -->

Quando [situacao real], quero [acao/capacidade], para [resultado].

**Exemplo**: "Quando partilho um link encurtado, quero que o visitante seja reencaminhado quase instantaneamente, para nao perder cliques."

---

## 2. Porque preciso disto?

[Problema que resolve ou beneficio que traz. Omitir se trivial.]

---

## 3. Criterios de Aceitacao — o contrato de "funciona"

> O coracao do pedido. Cada criterio (CA) e um cenario **verificavel** em linguagem de negocio.
> **Regras**: **1 CA = 1 obrigacao** (nao misturar). Os **erros / edge cases** e os **requisitos nao-funcionais que bloqueiam** (performance, seguranca, idioma/formato local) sao **CA proprios**, nao notas. Marcar `[bloqueia-close]` nos que travam o fecho.
> Cada CA carrega uma linha **Prova** — *o que demonstra que passou*. Sem prova executavel, o CA e vago: reescrever. (O detalhe tecnico da prova vive na spec; aqui basta o tipo + os dados + onde se observa.)

### CA-01 — [nome curto]  [bloqueia-close]
- **Dado** [estado inicial completo]
- **Quando** [uma acao]
- **Entao** [um resultado observavel e mensuravel — unidade, codigo de erro, header, valor]
- **Prova**: [teste automatico | oraculo de regressao | benchmark | smoke manual] · [fixture/dados] · [onde se observa]

### CA-02 — [nome curto]  [bloqueia-close]
- **Dado** ... **Quando** ... **Entao** ...
- **Prova**: ...

<!-- ... tantos quantos forem precisos. -->

> **Calculo regulado / com valor legal** (resultado tem de bater uma referencia): acrescentar um CA de **nao-regressao** — "toda a suite de oraculos de referencia mantem-se dentro da tolerancia aprovada; qualquer diferenca exige aprovacao do dono". Oraculos identificados por **ID/versao estaveis** (nao por nome). No veredicto que cruza um limite legal, a tolerancia e **zero**.

---

## 4. Dependencias

> Secao propria (nao "nota"): se uma dependencia falha, o pedido falha. Escrever "Nenhuma" se nao houver.

| Dependencia | Estado | Owner | Como testar se indisponivel |
|-------------|--------|-------|-----------------------------|
| [ex.: auth-service (JWT)] | DONE / em curso | [quem] | [mock / stub / fixture] |

---

## 5. Fora de Ambito (explicito)

- **Excluido**: [nao entra de todo]
- **Preparado mas nao implementado / divida conhecida**: [...] (→ pedido separado, se aplicavel)

---

## 6. Questoes em aberto

> Cada uma diz o que **bloqueia**.

- [ ] [Questao] — **bloqueia**: request | spec | implement | check · **owner**: [quem]

---

## 7. Notas (so contexto NAO-normativo)

<!-- Nada que bloqueie close vive aqui. Se bloqueia → e CA, dependencia, ou fora-de-ambito. -->

[Contexto de fundo, referencias, valores tipicos. Opcional.]

---

<!--
NOTA PARA A EQUIPA TECNICA:
Este pedido sera convertido num spec.md tecnico via /specify.
Cada CA deste pedido mapeia para um RF na spec (RF ↔ CA bidirecional: nenhum CA sem RF, nenhum RF sem CA).
A linha "Prova" de cada CA torna-se a verificacao executada em /sdd-check e listada em /sdd-close.

VARIANTE BUG (defeito em funcionalidade existente — vai direto ao tracker, sem pasta de request):
  ## Reproducao: passos + ambiente
  ## Atual vs esperado
  ## CA-01 — Nao reproduz [bloqueia-close]: Dado <repro> Quando <acao> Entao <esperado>
     Prova: teste de regressao que falha hoje e passa depois da correcao
-->
