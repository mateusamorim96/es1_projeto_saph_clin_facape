# CT-G03-03 — Leito negado por G4 preserva o motivo e a ficha

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G03-03 |
| **Requisito rastreado** | RF-G03-11 |
| **Caso de uso** | UC-G03-03 (FE-01) |
| **Tipo** | exceção |
| **Prioridade** | alta |

## Objetivo

Garante que a negativa de G4 é registrada com o motivo, que nada da ficha se perde e que o regulador pode decidir o desvio.

## Pré-condições

- Ficha em `EM_PREENCHIMENTO` com um `RegistroSinaisVitais` e um `Procedimento`.
- `PreAdmissao` enviada (`ENVIADA`), ficha em `AGUARDANDO_LEITO`.

## Dados de entrada

| Campo | Valor |
| --- | --- |
| resposta de G4 | `negado` |
| motivo | `LEITO_INDISPONIVEL` |

## Passos

1. G4 devolve `negado` com o motivo.
2. Consulta a `PreAdmissao` e o `Prontuario`.
3. O regulador registra uma nova teleorientação (UC-G03-01) ou abre um novo pedido.

## Resultado esperado

- `PreAdmissao` fica `NEGADA` com `motivo_negativa = LEITO_INDISPONIVEL`, visível ao regulador.
- `Prontuario` volta a `EM_PREENCHIMENTO`.
- Os registros clínicos continuam intactos; a prioridade não muda.
- O regulador consegue seguir com uma das duas opções do passo 3.

## Resultado obtido

não executado

## Critério de aceite

Passa se o motivo é gravado e visível, a ficha volta a `EM_PREENCHIMENTO`, nenhum registro é perdido e a prioridade permanece igual. Falha em qualquer outro caso.

## Observações

A resposta de sucesso (`reservado`) é premissa P4; o contrato só descreve o `negado`. A opção de "novo pedido a outro hospital" depende da definição de G4.
