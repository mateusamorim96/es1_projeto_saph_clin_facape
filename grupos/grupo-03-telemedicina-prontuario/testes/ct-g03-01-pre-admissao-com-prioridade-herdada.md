# CT-G03-01 — Pré-admissão sai com a prioridade herdada do chamado

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G03-01 |
| **Requisito rastreado** | RF-G03-09, RF-G03-10 |
| **Caso de uso** | UC-G03-03, UC-G03-05 |
| **Tipo** | funcional (fluxo principal) |
| **Prioridade** | alta |

## Objetivo

Garante que o pedido C3 sai com os seis campos obrigatórios e com a `prioridade` idêntica à do chamado, sem nenhum caminho para editá-la no G3.

## Pré-condições

- Despacho ativo e ficha em `EM_PREENCHIMENTO`.
- Um `RegistroSinaisVitais` gravado às 14:02.
- `prioridade_herdada` = `LARANJA` (recebida de G1 por C2, ADR-G03-01).
- Conexão disponível; G4 responde `reservado`.

## Dados de entrada

| Campo | Valor |
| --- | --- |
| `necessidade` | `SALA_VERMELHA` |
| `eta_hospital` | 8 min |
| `prioridade` (leitura) | `LARANJA` |

## Passos

1. O médico regulador abre a ficha e aciona "Solicitar pré-admissão".
2. Tenta alterar o campo de prioridade na tela do resumo.
3. Informa `necessidade` e `eta_hospital` e confirma o envio.
4. G4 responde `reservado`.
5. Consulta o estado da `PreAdmissao` e do `Prontuario`.

## Resultado esperado

- O passo 2 não permite edição: a prioridade é exibida somente leitura.
- O C3 publicado tem `prontuario_id`, `chamado_id`, `prioridade = LARANJA`, `necessidade`, último `sinais_vitais` e `eta_hospital`.
- `PreAdmissao` fica `CONFIRMADA` e `Prontuario` fica `LEITO_CONFIRMADO`.

## Resultado obtido

não executado

## Critério de aceite

Passa se os seis campos do C3 estão presentes, a `prioridade` é igual à do chamado e não há como editá-la. Falha em qualquer outro caso.

## Observações

Depende do ADR-G03-01 (prioridade no C2), ainda `proposto`. Com o plano B (consulta a G1), o resultado esperado é o mesmo.
