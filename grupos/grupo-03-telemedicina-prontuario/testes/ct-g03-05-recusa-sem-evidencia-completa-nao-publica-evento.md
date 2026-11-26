# CT-G03-05 — Recusa sem as três evidências não publica RECUSA_ASSINADA

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G03-05 |
| **Requisito rastreado** | RF-G03-12, RF-G03-13 |
| **Caso de uso** | UC-G03-04 (FE-01, FE-02, FA-02) |
| **Tipo** | exceção |
| **Prioridade** | alta |

## Objetivo

Garante que só a recusa com assinatura, testemunha e ciência do regulador (RN-10) vira `ASSINADA` e gera evento.

## Pré-condições

- Ficha em `EM_PREENCHIMENTO`; paciente avaliado pela equipe.
- Cinco cenários, um por execução: (a) consciência não confirmada; (b) falta a assinatura do paciente; (c) falta a testemunha; (d) falta a ciência do regulador; (e) as três evidências presentes.

## Dados de entrada

| Cenário | Situação |
| --- | --- |
| a | `consciencia_confirmada = falso` |
| b | paciente se recusa a assinar; testemunha e ciência presentes |
| c | sem testemunha |
| d | sem ciência do regulador |
| e | assinatura do paciente, testemunha e ciência |

## Passos

1. A equipe inicia "Registrar recusa de atendimento".
2. Conduz o cenário até o ponto de conclusão.
3. Consulta o estado da `RecusaAtendimento`, o da ficha e os eventos publicados.

## Resultado esperado

- (a) a recusa não é criada.
- (b) estado `NAO_ASSINADA`, sem evento `RECUSA_ASSINADA`, com alerta ao regulador.
- (c) e (d) permanece `RASCUNHO`, com a evidência faltante indicada, sem evento.
- (e) estado `ASSINADA`, ficha `RECUSADO` e um evento `RECUSA_ASSINADA` com a `justificativa` preenchida.

## Resultado obtido

não executado

## Critério de aceite

Passa se o evento só existe no cenário (e), exatamente uma vez. Falha se algum cenário de (a) a (d) publicar o evento.

## Observações

O estado `NAO_ASSINADA` é proposta do grupo (ADR-G03-04, ainda `proposto`).
