# Pedido: [Nome curto e concreto da funcionalidade]

> **Data**: [DATA] · **Pedido por**: [NOME] · **Prioridade**: Alta / Media / Baixa
> **Capacidade-mae**: [ID estavel da capacidade no Mapa Funcional, ex.: `CAP-<APP>-<EPIC>-<NNN>`]

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
