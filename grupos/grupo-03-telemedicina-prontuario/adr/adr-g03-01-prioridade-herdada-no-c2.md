# ADR-G03-01 — A prioridade chega ao G3 no contrato C2 e é somente leitura

## Status

`proposto` (depende de aceite de G1 e G2, pois altera o contrato C2)

Data: 2026-11-26
Decisores: Luccas, Tulio, Geliton

## Contexto

O contrato C3 `pedido_pre_admissao` exige `prioridade` "herdada da triagem, não recalculada"
(`arquitetura-modular.md`, C3). A prioridade clínica é do G1 (regra de coerência 4; RN-02), mas o contrato
C2 `despacho_ativo`, único que traz dados ao G3, tem só `despacho_id`, `chamado_id`, `equipe_id` e
`eta_local`. A arquitetura também proíbe depender de chamada síncrona entre módulos (§6). Um chamado pode
ser reclassificado depois do despacho (RN-02), então a prioridade que o G3 usa precisa acompanhar G1 sem
que o G3 a recalcule. Sem decisão, o RF-G03-10 não tem fonte para o campo que ele valida (questão Q1).

## Decisão

O G3 trata a prioridade como valor herdado e somente leitura, recebido no C2. Propõe-se que o C2 passe a
carregar `prioridade` e `prioridade_atualizada_em`, e que cada reclassificação em G1 reemita o C2 (mesmo
`despacho_id`, nova versão) por intermédio de G2. O `Prontuario` guarda `prioridade_herdada` e
`prioridade_atualizada_em`; a `PreAdmissao` copia o valor vigente no momento do envio.

## Alternativas consideradas

| Alternativa | Prós | Contras | Por que foi descartada |
| --- | --- | --- | --- |
| A. G3 consulta G1 por `chamado_id` ao validar | Não muda contrato | Chamada síncrona entre módulos (ARQ §6); falha se G1 estiver indisponível | Acopla o envio da pré-admissão à disponibilidade de G1. Fica como plano B. |
| B. G3 recalcula a prioridade com os sinais vitais | Independe de outros módulos | Viola RN-02 e a regra de coerência 4; duas prioridades divergentes | Prioridade clínica é do G1. |
| C. Regulador digita a prioridade na ficha | Simples | Duplica o dado e permite divergir do chamado | Quebra a regra "herdada, não recalculada". |

## Consequências

**Positivas**

- O RF-G03-10 tem fonte clara e a prioridade nunca é editada no G3.
- A pré-admissão funciona mesmo com G1 fora do ar, usando o último valor recebido.

**Negativas / custos**

- G2 passa a reemitir o C2 a cada reclassificação.
- O G3 pode usar uma prioridade defasada por alguns segundos; `prioridade_atualizada_em` torna isso visível.

**Impactos em outros grupos**

- G1 e G2: o contrato C2 ganha dois campos (`prioridade`, `prioridade_atualizada_em`). Mudança de contrato exige menção no PR dos grupos afetados.
- Se G1/G2 recusarem, adota-se a alternativa A como plano B e este ADR é substituído.

## Como verificar

- Diagrama de classes: `Prontuario.prioridade_herdada` marcada `{somente leitura}`; `PreAdmissao.prioridade` marcada `{cópia no envio}`.
- DS-G03-02: a validação (`<<include>>` UC-G03-05) lê a prioridade do `Prontuario`, sem recalcular.
- Matriz de rastreabilidade, seção de contratos consumidos: linha do C2 com o campo `prioridade`.
- Caso de teste (M3): pedido enviado com `prioridade` igual à do chamado, e nenhum caminho de edição.
