# ADR-G03-02 — Registros clínicos são imutáveis e versionados

## Status

`aceito`

Data: 2026-11-26
Decisores: Luccas, Tulio, Geliton

## Contexto

A equipe corrige dados em campo (valor digitado errado, horário trocado), e o RF-G03-04 exige que o
histórico dos sinais vitais seja mantido. Toda ação sensível gera evento imutável (RN-09), e a norma de
telemedicina trata a ficha como documento com integridade e rastreabilidade (Res. CFM 2.314/2022, art. 3º,
conforme leitura do grupo; a redação vigente deve ser conferida). Se um registro puder ser sobrescrito, o
valor que embasou uma conduta clínica se perde, e a prova de quem registrou o quê também.

## Decisão

Um `RegistroClinico` nunca é alterado nem apagado. A correção cria um novo registro com `versao` + 1,
`substitui_id` apontando a versão anterior e `motivo_correcao` obrigatório. A versão original permanece
no histórico. Consultas e o C3 usam a versão vigente (a última da cadeia).

## Alternativas consideradas

| Alternativa | Prós | Contras | Por que foi descartada |
| --- | --- | --- | --- |
| A. Atualizar o registro no lugar, com log de alterações | Modelo simples | O log vira trilha paralela (regra de coerência 5); o valor original some da ficha | Duplica a função do G6 e perde o histórico clínico. |
| B. Exclusão lógica (marcar como removido) | Preserva o dado | Não liga o valor corrigido ao original nem registra o motivo | Não permite reconstruir a sequência de correções. |
| C. Versionamento por `substitui_id` (escolhida) | Histórico completo, motivo explícito, funciona com a sincronização | Mais registros e consulta pela última versão | — |

## Consequências

**Positivas**

- Nenhum valor clínico se perde, o que ajuda a cumprir o RNF-G03-02 (0 registros perdidos).
- A herança `RegistroClinico` concentra autoria, horários e versionamento para as três subclasses.

**Negativas / custos**

- Consultas precisam escolher a versão vigente; o dimensionamento do RNF-G03-01 conta só as versões vigentes (30 + 20 + 10).
- Quem corrige precisa informar o motivo, o que adiciona um passo em campo (mitigar no wireframe do M3).

**Impactos em outros grupos**

- Nenhum contrato muda. O C3 e o C4 sempre carregam a versão vigente.

## Como verificar

- Diagrama de classes: `RegistroClinico.versao`, `substitui_id` e `motivo_correcao`; herança das três subclasses.
- Dicionário de dados, seção 2.2: restrição "versão > 1 exige `substitui_id` e `motivo_correcao`".
- Caso de teste (M3): corrigir um sinal vital preserva o valor original, o autor e o motivo (FA-04 do UC-G03-02).
