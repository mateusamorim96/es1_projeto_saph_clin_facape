# ADR-G03-05 — A teleorientação se vincula ao chamado e o despacho é opcional

## Status

`proposto` (depende de G2 criar um contrato novo para o caso sem viatura)

Data: 2026-11-26
Decisores: Luccas, Tulio, Geliton

## Contexto

A definição do projeto prevê desviar o atendimento para a telemedicina quando não há viatura disponível,
e o contexto pergunta como esse desvio é registrado. Em `temas-grupos.md`, a entrada do G3 é o despacho
ativo, e o único contrato que chega ao G3 é o C2 `despacho_ativo`. Logo, sem viatura não há C2 e a
teleorientação ficaria sem origem. A teleorientação é atribuição do G3, e o chamado pertence ao G1 (questão Q3).

## Decisão

A `Teleorientacao` exige `chamado_id` e `medico_id`; o `despacho_id` é opcional. Quando não há despacho, o
G3 marca `desvio_sem_viatura = verdadeiro` e vincula a orientação só ao chamado. Propõe-se um contrato novo
de G2 para G3, `sem_viatura_disponivel`, com `chamado_id` e motivo, que coloca o chamado na fila de
regulação do G3. Quando um despacho chegar depois, a ficha é aberta e passa a exibir as teleorientações
anteriores do mesmo `chamado_id`.

## Alternativas consideradas

| Alternativa | Prós | Contras | Por que foi descartada |
| --- | --- | --- | --- |
| A. Exigir despacho ativo | Não muda contrato | Impede o desvio previsto na definição do projeto | Deixaria sem solução justamente o caso de indisponibilidade de viatura. |
| B. G3 cria um chamado próprio | Independe de G1 | Duplica a entidade `Chamado` de G1 | Quebra a regra de coerência 1. |
| C. Mover a teleorientação sem despacho para o G1 | Sem contrato novo em G3 | Contradiz o tema do G3, onde a teleorientação é do módulo | Dividiria a mesma funcionalidade entre dois grupos. |
| D. Chamado obrigatório e despacho opcional (escolhida) | Cobre o desvio; mantém dono do chamado | Exige contrato novo de G2 | — |

## Consequências

**Positivas**

- O desvio para a telemedicina passa a ter caminho registrado e auditável (evento `TELEORIENTACAO_REGISTRADA`).
- O modelo continua sem duplicar o chamado.

**Negativas / custos**

- `Teleorientacao.prontuario_id` fica opcional: a ficha pode não existir quando a orientação é dada.
- O G3 precisa de uma fila de regulação alimentada por G2 para esse caso.

**Impactos em outros grupos**

- G2: novo contrato `sem_viatura_disponivel` (G2 → G3). Mudança de contrato exige menção no PR dos grupos afetados.
- G1: nenhum campo novo; o G3 só referencia o `chamado_id`.

## Como verificar

- Diagrama de classes: `Teleorientacao.despacho_id [0..1]`, `desvio_sem_viatura` e multiplicidade `Prontuario 0..1 — 0..* Teleorientacao`.
- UC-G03-01, fluxo alternativo FA-02.
- DS-G03-03: ramo de desvio que leva a uma nova teleorientação.
- Caso de teste (M3): registrar teleorientação com chamado válido e sem despacho.
