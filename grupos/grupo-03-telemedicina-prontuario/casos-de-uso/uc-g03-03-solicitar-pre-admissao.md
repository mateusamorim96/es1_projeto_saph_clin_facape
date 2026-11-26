# UC-G03-03 — Solicitar Pré-Admissão

> **Fluxo crítico do módulo.** Corresponde ao fluxo "Pré-Alocação de Leito e Passagem de Plantão Digital"
> da definição do projeto (Viatura → Sistema → Sala Vermelha), na parte que pertence ao G3.

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | UC-G03-03 |
| **Módulo** | G03 — Telemedicina e Prontuário Pré-Hospitalar |
| **Ator primário** | MedicoRegulador |
| **Atores secundários** | EquipeIntervencao (fornece os dados clínicos), Módulo de Leitos (G4), EquipeReceptora (recebe o alerta emitido por G4), Módulo de Auditoria (G6) |
| **Objetivo** | Enviar ao hospital, antes da chegada, o resumo clínico e a necessidade de leito, para que G4 reserve o leito |
| **Requisitos rastreados** | RF-G03-09, RF-G03-10, RF-G03-11, RNF-G03-06 |
| **Prioridade** | essencial |

## Pré-condições

- Existe despacho ativo e ficha aberta para o chamado.
- A ficha tem ao menos um `RegistroSinaisVitais`.
- A prioridade do chamado é conhecida por este módulo (ver Q1 no README: C2 não a traz).
- A ficha não tem recusa de atendimento assinada (UC-G03-04).

## Pós-condições

- **Sucesso:** `PreAdmissao` criada e enviada; contrato C3 `pedido_pre_admissao` publicado para G4. Quando G4 reserva o leito, a pré-admissão fica "confirmada" e a ficha passa a ser consultável pela `EquipeReceptora`.
- **Falha:** `PreAdmissao` em "negada", "pendente de envio" ou "sem resposta", conforme o fluxo de exceção. A ficha é preservada e nenhuma prioridade é alterada.

## Fluxo principal

1. O médico regulador abre a ficha do atendimento e aciona "Solicitar pré-admissão".
2. O sistema exibe o resumo: prioridade (somente leitura), último registro de sinais vitais, procedimentos e fármacos já registrados.
3. O médico regulador informa a necessidade de leito: `SALA_VERMELHA` ou `OBSERVACAO`.
4. O médico regulador informa ou confirma o tempo estimado de chegada ao hospital (`eta_hospital`).
5. O sistema valida a prioridade e a completude do pedido (**`<<include>>` UC-G03-05**).
6. O médico regulador confirma o envio.
7. O sistema cria a `PreAdmissao` em "enviada" e publica o C3 para G4, com `prontuario_id`, `chamado_id`, `prioridade`, `necessidade`, `sinais_vitais` e `eta_hospital`.
8. G4 responde que o leito foi reservado.
9. O sistema atualiza a `PreAdmissao` para "confirmada" e avisa o médico regulador e a equipe de intervenção.
10. O sistema libera a ficha consolidada para consulta da `EquipeReceptora` (UC-G03-07), evitando que a recepção redigite os dados.

## Fluxos alternativos

### FA-01 — Deterioração clínica após o envio

- **Condição:** o quadro piora e a necessidade muda (por exemplo, de `OBSERVACAO` para `SALA_VERMELHA`).
- **Passos:** 1. O regulador atualiza a necessidade e envia novo pedido com os sinais vitais mais recentes. 2. O sistema reexecuta a validação (UC-G03-05). 3. Se o chamado foi reclassificado em G1, o sistema passa a herdar a nova prioridade; este módulo não a recalcula (RN-02).
- **Reconvergência:** retorna ao passo 7 do fluxo principal.

### FA-02 — Equipe informa o ETA

- **Condição:** quem conhece o tempo de chegada é a equipe na viatura.
- **Passos:** 1. A equipe de intervenção informa o `eta_hospital` pela ficha. 2. O regulador confere o valor.
- **Reconvergência:** retorna ao passo 5 do fluxo principal.

## Fluxos de exceção

### FE-01 — Leito negado por G4

- **Condição:** G4 devolve `negado` com motivo (falha de contrato do C3).
- **Passos:** 1. O sistema grava a `PreAdmissao` como "negada" e exibe o motivo. 2. O sistema avisa o regulador. 3. O regulador decide o desvio: nova teleorientação (UC-G03-01) ou novo pedido, conforme o que o contrato permitir (ver Q4).
- **Estado final:** `PreAdmissao` "negada" com motivo preservado; ficha intacta.

### FE-02 — Validação falha

- **Condição:** falta `prioridade`, sinais vitais, `necessidade` ou `eta_hospital` no passo 5.
- **Passos:** 1. O sistema bloqueia o envio. 2. O sistema indica o campo faltante.
- **Estado final:** nenhuma `PreAdmissao` enviada; pedido permanece como rascunho.

### FE-03 — Sem conexão no momento do envio

- **Condição:** o dispositivo não consegue entregar o C3.
- **Passos:** 1. O sistema grava a `PreAdmissao` como "pendente de envio" e alerta de forma visível. 2. O sistema reenvia sozinho ao reconectar, com o mesmo `evento_id` (sem duplicar). 3. Enquanto isso, a equipe aciona a recepção por canal alternativo, fora do sistema.
- **Estado final:** pedido pendente, preservado e com o horário do fato original.

### FE-04 — G4 não responde no prazo

- **Condição:** nenhuma resposta de G4 dentro do prazo-limite (premissa do grupo: 60 s, a definir por ADR).
- **Passos:** 1. O sistema marca a `PreAdmissao` como "sem resposta". 2. O sistema alerta o regulador e oferece reenvio manual.
- **Estado final:** `PreAdmissao` "sem resposta"; o atendimento não é bloqueado.

## Regras de negócio aplicadas

| Regra | Descrição |
| --- | --- |
| RN-02 | FA-01: a prioridade não é sobrescrita; a reclassificação é de G1 |
| RN-05 e RN-06 | tratadas em G4; este módulo só consome a resposta |
| RN-09 | o envio é ação sensível (ver Q5 sobre o evento) |
| RN-11 | passo 10: a recepção só acessa a ficha após o pedido enviado |
| RN-12 | o horário do pedido alimenta o indicador porta-leito de G6 |

## Relacionamentos

- `<<include>>` **UC-G03-05 Validar Prioridade**: obrigatório. O C3 exige `prioridade` herdada, não recalculada (ARQ, C3), e G3 não pode alterá-la (regra 4 de coerência). A verificação está isolada porque também é reutilizada em FA-01.

## Dados manipulados

| Entidade | Dado | Origem |
| --- | --- | --- |
| Chamado | `chamado_id`, `prioridade` | referência a G1 |
| Prontuario | `prontuario_id` | este módulo |
| RegistroSinaisVitais | último registro | este módulo |
| PreAdmissao | `necessidade`, `eta_hospital`, estado | este módulo |
| Leito / Reserva | resultado da reserva | G4 (só por referência) |

## Critérios de aceite

- [ ] O C3 sai com os seis campos obrigatórios e `prioridade` igual à do chamado.
- [ ] Nenhum cenário permite editar `prioridade` neste módulo.
- [ ] Resposta `negado` é exibida com o motivo e não altera a ficha.
- [ ] Reenvio após queda de conexão não duplica o pedido.
- [ ] O C3 é publicado em ≤ 3 s (p95) após a confirmação do regulador, com conexão (RNF-G03-06).

## Rastreabilidade

| RF | Classe | Sequência | Teste |
| --- | --- | --- | --- |
| RF-G03-09 | pendente (M2) | pendente (M2) | pendente (M3) |
| RF-G03-10 | pendente (M2) | pendente (M2) | pendente (M3) |
| RF-G03-11 | pendente (M2) | pendente (M2) | pendente (M3) |
