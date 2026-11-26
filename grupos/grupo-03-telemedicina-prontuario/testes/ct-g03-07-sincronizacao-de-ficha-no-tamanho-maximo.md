# CT-G03-07 — Sincronização de uma ficha no tamanho máximo dimensionado

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G03-07 |
| **Requisito rastreado** | RNF-G03-01 |
| **Caso de uso** | UC-G03-08 |
| **Tipo** | fronteira / não funcional |
| **Prioridade** | média |

## Objetivo

Verifica o limite do RNF-G03-01: ficha de até 30 sinais vitais, 20 procedimentos e 10 fármacos sincroniza em até 5 s (p95) após a reconexão.

## Pré-condições

- Tablet offline com a ficha no tamanho máximo: 30 `RegistroSinaisVitais`, 20 `Procedimento` e 10 `ConsumoFarmaco`, cada um `PENDENTE` (60 registros).
- Conexão de referência definida no M3 (a mesma em todas as execuções).

## Dados de entrada

| Campo | Valor |
| --- | --- |
| tamanho da ficha | 30 + 20 + 10 registros |
| repetições | 20 execuções, para calcular o percentil 95 |
| limite | ≤ 5 s |

## Passos

1. Restabelecer a conexão e iniciar a contagem do tempo.
2. Aguardar o último registro ficar `SINCRONIZADO`.
3. Repetir 20 vezes e calcular o percentil 95.
4. Repetir com 31 sinais vitais (um acima do limite) e registrar o comportamento.

## Resultado esperado

- Percentil 95 do tempo ≤ 5 s com a ficha no limite.
- Com 31 sinais vitais, a sincronização ainda funciona e nenhum registro se perde; o tempo é só registrado, pois o requisito não promete prazo acima do limite.

## Resultado obtido

não executado

## Critério de aceite

Passa se p95 ≤ 5 s no limite e nenhum registro for perdido no caso acima dele. Falha se p95 > 5 s.

## Observações

O tamanho da ficha é premissa do grupo (RNF-G03-01) e deve ser revisado com os dados reais do M3.
