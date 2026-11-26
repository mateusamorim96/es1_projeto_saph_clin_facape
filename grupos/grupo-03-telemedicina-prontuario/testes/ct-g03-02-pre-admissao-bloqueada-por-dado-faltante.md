# CT-G03-02 — Pré-admissão é bloqueada quando falta dado obrigatório

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G03-02 |
| **Requisito rastreado** | RF-G03-10 |
| **Caso de uso** | UC-G03-05, UC-G03-03 (FE-02) |
| **Tipo** | exceção |
| **Prioridade** | alta |

## Objetivo

Garante que nenhum C3 incompleto é enviado a G4 e que o campo faltante é indicado.

## Pré-condições

- Despacho ativo; ficha em `EM_PREENCHIMENTO`.
- Quatro cenários, um por execução: (a) sem nenhum `RegistroSinaisVitais`; (b) sem `necessidade`; (c) sem `eta_hospital`; (d) sem prioridade vigente disponível.

## Dados de entrada

| Cenário | Campo ausente |
| --- | --- |
| a | sinais vitais |
| b | `necessidade` |
| c | `eta_hospital` |
| d | `prioridade_herdada` |

## Passos

1. O médico regulador aciona "Solicitar pré-admissão" e tenta confirmar o envio.
2. Observa a resposta do sistema.
3. Consulta o estado da `PreAdmissao` e verifica se algum C3 foi publicado.

## Resultado esperado

- Em cada cenário, o envio é bloqueado e a tela indica o campo faltante.
- A `PreAdmissao` permanece `RASCUNHO`; nenhum C3 é publicado; a ficha continua `EM_PREENCHIMENTO`.

## Resultado obtido

não executado

## Critério de aceite

Passa se, nos quatro cenários, não há C3 publicado e o campo faltante é indicado. Falha se qualquer cenário enviar o pedido.

## Observações

No cenário (d), a prioridade nunca é preenchida manualmente para destravar o envio (ADR-G03-01).
