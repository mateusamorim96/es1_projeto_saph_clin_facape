# ADR-G03-04 — A recusa de atendimento é uma entidade com estados e exige três evidências

## Status

`proposto` (depende de G6 expor o registro de assinatura e de confirmação do professor sobre a RN-10)

Data: 2026-11-26
Decisores: Luccas, Tulio, Geliton

## Contexto

A RN-10 diz que a recusa de paciente consciente exige assinatura e testemunha, mas não diz de quem é a
assinatura. A definição do projeto, no módulo de auditoria, cita a assinatura do médico regulador em
recusas de pacientes conscientes. A assinatura digital é entidade do G6 (regra de coerência 1), e o G3 não
pode duplicá-la. A RN-10 também não prevê o paciente que se recusa a assinar. Essas lacunas são a questão Q4.

## Decisão

A `RecusaAtendimento` é uma entidade própria do G3, com estados `RASCUNHO`, `ASSINADA`, `NAO_ASSINADA` e
`REVOGADA`. Somente `ASSINADA` satisfaz a RN-10 e publica `RECUSA_ASSINADA`, e exige três evidências:
assinatura do paciente (ou do representante legal), testemunha identificada e ciência do médico regulador.
O G3 guarda apenas as referências `assinatura_*_id` de G6. O estado `NAO_ASSINADA` registra a recusa com
testemunha e ciência quando o paciente não quer assinar, não satisfaz a RN-10 e alerta o regulador. A recusa só
pode ser criada com a consciência do paciente confirmada. Ficha com recusa `ASSINADA` não oferece pré-admissão.

## Alternativas consideradas

| Alternativa | Prós | Contras | Por que foi descartada |
| --- | --- | --- | --- |
| A. Atributo booleano `recusou` no `Prontuario` | Simples | Não prova a RN-10, não guarda evidência nem estado | Sem evidência, a recusa não tem valor probatório. |
| B. Guardar a imagem da assinatura no G3 | Sem dependência de G6 | Duplica a `AssinaturaDigital` de G6 | Quebra a regra de coerência 1. |
| C. Exigir só a assinatura do médico regulador | Segue a definição à risca | Não evidencia a manifestação do paciente | Não cobre a assinatura do paciente que a RN-10 pressupõe. |
| D. Entidade com três evidências (escolhida) | Cobre as duas leituras; rastreável | Mais passos em campo; depende de G6 | — |

## Consequências

**Positivas**

- A recusa tem estados claros e o evento só sai quando a RN-10 está cumprida.
- Nenhuma assinatura é duplicada: o G3 só referencia o `assinatura_id`.

**Negativas / custos**

- Mais passos para a equipe em campo (mitigar no wireframe do M3).
- O estado `NAO_ASSINADA` é proposta do grupo, não regra do professor. Precisa de confirmação.

**Impactos em outros grupos**

- G6: precisa oferecer uma operação de registro de assinatura (`registrarAssinatura`) que devolva `assinatura_id`. Não há esse contrato na arquitetura; é um contrato novo proposto, a combinar com o G6.

## Como verificar

- Diagrama de classes: `RecusaAtendimento` com três `assinatura_*_id` «G6» e `estado`.
- ME-G03-03 (estados) e DS-G03-05 (sequência com os ramos de evidência incompleta e de não assinatura).
- DA-G03-02: decisões de consciência, assinatura e evidências.
- Casos de teste (M3): recusa sem uma das três evidências não publica o evento.
