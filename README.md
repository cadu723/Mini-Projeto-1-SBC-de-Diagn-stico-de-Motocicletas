# Mini-Projeto 1 — SBC de Diagnóstico de Motocicletas

Disciplina: Sistemas Baseados em Conhecimento (SBC) — Prof. Daniel Faustino
Equipe: NOME 1 · NOME 2

## Domínio

O sistema é um Sistema Baseado em Conhecimento de regras IF–THEN que ajuda a diagnosticar problemas comuns em motocicletas a partir de sintomas relatados pelo piloto ou observados pelo mecânico (quilometragem do óleo, barulho do motor, folga da corrente, estado do freio, som da partida etc.) e recomenda uma ação.

O raciocínio é feito por encadeamento progressivo (forward chaining) com a biblioteca `experta`, em três níveis:

1. **Sintoma → Estado do componente** (ex.: óleo com 2100 km → óleo vencido)
2. **Estado → Diagnóstico** (ex.: óleo vencido + barulho metálico → desgaste acelerado)
3. **Diagnóstico → Ação recomendada** (ex.: desgaste + vazamento → chamar guincho)

Fatos usados: `Sintoma` (entrada), `EstadoComponente`, `Diagnostico` e `AcaoRecomendada`.

## Regras em linguagem natural

**Nível 1 — Sintoma → Estado**

- **R1:** SE o óleo tem 2000 km ou mais desde a troca ENTÃO o óleo está vencido.
- **R2:** SE o óleo tem menos de 2000 km ENTÃO o óleo está bom.
- **R3:** SE a folga da corrente é de 3,0 cm ou mais ENTÃO a corrente está frouxa.
- **R4:** SE a partida faz som de cliques ENTÃO a bateria está fraca.
- **R5 (salience = 100):** SE o freio está borrachudo ENTÃO emitir alerta crítico e recomendar troca imediata do freio.

**Nível 2 — Estado → Diagnóstico**

- **R6:** SE o óleo está vencido E há barulho metálico no motor ENTÃO há desgaste acelerado.
- **R7:** SE o óleo está vencido E NÃO há barulho metálico ENTÃO é apenas troca de rotina.
- **R8:** SE a corrente está frouxa E a marcha está dura ENTÃO o kit relação está ressecado.
- **R9:** SE a bateria está fraca ENTÃO a bateria está sem carga.

**Nível 3 — Diagnóstico → Ação**

- **R10:** SE há desgaste acelerado E há vazamento ENTÃO chamar guincho (risco de fundir o motor).
- **R11:** SE há desgaste acelerado E NÃO há vazamento ENTÃO levar ao mecânico devagar, trocar o óleo e verificar o cilindro.
- **R12:** SE é troca de rotina ENTÃO fazer manutenção preventiva (troca de óleo 10W-30 semissintético).
- **R13:** SE o kit relação está ressecado ENTÃO esticar a corrente e lubrificar.

## Resolução de conflito

- **Salience:** a R5 (freio borrachudo) tem `salience=100`. Quando ela está na agenda junto com outras regras, o motor a dispara primeiro, porque freio é risco de vida.
- **NOT:** as regras R7 e R11 usam `NOT(...)` para separar casos mutuamente exclusivos (com/sem barulho metálico, com/sem vazamento), evitando diagnósticos concorrentes.

## Explicabilidade

Cada regra imprime um log `[LOG] Regra N ativada` com o motivo, e cada ação final mostra o trace da cadeia que levou até ela (ex.: "Decisão tomada porque as Regras 1, 6 e 10 dispararam em cadeia").

## Casos de teste

| Caso | Entrada principal | Regras disparadas | Saída esperada |
|---|---|---|---|
| 1 | óleo 2100 km, barulho metálico, sem vazamento, corrente 3,5 cm, marcha dura | R1 → R6 → R11 e R3 → R8 → R13 | Levar ao mecânico devagar + esticar e lubrificar a corrente |
| 2 | óleo 2500 km, barulho metálico, com vazamento, corrente 4,5 cm, marcha dura | R1 → R6 → R10 e R3 → R8 → R13 | Chamar guincho + esticar e lubrificar a corrente |
| 3 | freio borrachudo, partida com cliques, óleo 1500 km | R5 (primeiro, por salience), depois R4 → R9 e R2 | Alerta crítico do freio aparece antes de tudo |

## Como executar

1. Abra `sbc_entregavel_1.ipynb` no Google Colab.
2. Menu **Ambiente de execução → Executar tudo**.
3. A primeira célula instala `experta` e atualiza o `frozendict` (necessário no Python 3.12 do Colab).
