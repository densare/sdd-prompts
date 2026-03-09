# C2. Areas e Volumes

> **Data**: 2026-02-10
> **Pedido por**: Nuno Januario
> **Prioridade**: Alta

---

## O que preciso?

Quero introduzir as areas e volumes da fracao: area util, pe direito, volume. Estes valores sao essenciais para os calculos termicos.

---

## Porque preciso disto?

A area util (Ap) e o volume (V) sao usados em todos os calculos termicos REH — Nic, Nvc, Ntc, Qa, fator de forma, ganhos internos. Sem estes valores preenchidos, nenhum calculo pode correr. E o segundo passo funcional apos a identificacao do processo.

---

## Como imagino que funcione?

1. Clico em "Areas e Volumes" sob os dados da fracao
2. Preencho:
   - Area util de pavimento (Ap)
   - Pe direito medio
   - Volume interior (ou calcula automatico a partir de area x pe direito)
3. O programa calcula Ap e V automaticamente

---

## O que deve aparecer no resultado?

Os valores introduzidos e os valores calculados (Ap, V). Se eu introduzo area e pe-direito, o volume calcula sozinho. Se eu introduzo o volume directamente, o pe-direito ajusta.

---

## Informacao adicional (opcional)

- Area bruta (Ag) e opcional mas util para o fator de forma
- Pe-direito por defeito: 2.7m (valor tipico em Portugal)
- Estes dados alimentam o motor de calculo REH (GeometricProperties)

---

<!--
NOTA PARA A EQUIPA TECNICA:
Este pedido sera analisado e convertido num spec.md tecnico.
Usar /specify dentherm C2 para iniciar a analise.

EXEMPLO: Este e um request real da task C2-areas-volumes do DenTherm.
Notar como usa linguagem simples, sem termos tecnicos de implementacao.
-->
