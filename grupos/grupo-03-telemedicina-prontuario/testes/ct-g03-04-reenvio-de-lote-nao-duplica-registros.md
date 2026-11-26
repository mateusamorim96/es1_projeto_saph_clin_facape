# CT-G03-04 — Reenviar o mesmo lote de registros não duplica nada

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G03-04 |
| **Requisito rastreado** | RF-G03-08, RF-G03-07, RNF-G03-02 |
| **Caso de uso** | UC-G03-08, UC-G03-02 |
| **Tipo** | fronteira (duplicidade e queda de conexão) |
| **Prioridade** | alta |

## Objetivo

Garante que a sincronização é idempotente e que uma nova queda durante o envio não perde nem duplica registros.

## Pré-condições

- Tablet offline com 3 registros `PENDENTE`: sinais vitais (14:02), procedimento (14:05) e sinais vitais (14:10).
- Ficha existente na central, sem esses registros.

## Dados de entrada

| Campo | Valor |
| --- | --- |
| lote | 3 registros, cada um com `registro_id` próprio |
| tentativas de envio | 3 envios do mesmo lote |
| queda | conexão cai após o 2º registro, no 1º envio |

## Passos

1. A conexão volta e o tablet inicia a sincronização; a conexão cai após o 2º registro.
2. A conexão volta de novo e o tablet reenvia o lote completo.
3. O tablet envia o mesmo lote uma terceira vez.
4. Consulta os registros da ficha na central e o estado de cada um no tablet.

## Resultado esperado

- A central tem exatamente 3 registros, sem cópias, na ordem do horário do fato (14:02, 14:05, 14:10).
- O 3º registro, não confirmado na queda, segue `PENDENTE` até o 2º envio e termina `SINCRONIZADO`.
- Nenhum registro é descartado em nenhum momento.

## Resultado obtido

não executado

## Critério de aceite

Passa se a central tem 3 registros únicos, na ordem correta, e os 3 estão `SINCRONIZADO` no tablet. Falha se houver cópia, perda ou ordem trocada.

## Observações

Cobre os cenários de queda "durante a sincronização" e "reenvio" do RNF-G03-02. ADR-G03-03.
