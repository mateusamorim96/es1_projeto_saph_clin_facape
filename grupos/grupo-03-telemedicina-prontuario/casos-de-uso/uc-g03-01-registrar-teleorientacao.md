# UC-G03-01 — Registrar Teleorientação

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | UC-G03-01 |
| **Módulo** | G03 — Telemedicina e Prontuário Pré-Hospitalar |
| **Ator primário** | MedicoRegulador |
| **Atores secundários** | EquipeIntervencao (destinatária da orientação), Módulo de Auditoria (G6) |
| **Objetivo** | Registrar a orientação dada pelo médico regulador antes da chegada da viatura, de forma rastreável e auditável |
| **Requisitos rastreados** | RF-G03-01, RF-G03-02, RNF-G03-05 |
| **Prioridade** | essencial |

## Pré-condições

- Existe um chamado classificado, identificado por `chamado_id` (origem: G1).
- Existe despacho ativo para o chamado (contrato C2, de G2), salvo o fluxo FA-02.
- O usuário está autenticado com perfil `MedicoRegulador`.

## Pós-condições

- **Sucesso:** `Teleorientacao` gravada com autor, horário do fato, destinatário e conteúdo. Evento
  `TELEORIENTACAO_REGISTRADA` publicado para G6. A orientação aparece na ficha pré-hospitalar do chamado.
- **Falha:** nenhum registro parcial é gravado. O regulador permanece na tela com o motivo da falha.

## Fluxo principal

1. O médico regulador seleciona, na fila de regulação, o chamado que receberá a orientação.
2. O sistema exibe os dados de identificação do chamado e do despacho recebidos por contrato, e as teleorientações anteriores do mesmo chamado.
3. O médico regulador informa o destinatário da orientação.
4. O médico regulador descreve a orientação dada.
5. O médico regulador confirma o registro.
6. O sistema verifica que destinatário e conteúdo estão preenchidos.
7. O sistema grava a teleorientação com o autor, o horário do fato e o horário do registro, e a indica como emitida por telemedicina.
8. O sistema publica o evento `TELEORIENTACAO_REGISTRADA` para G6.
9. O sistema torna a orientação visível na ficha pré-hospitalar do chamado (UC-G03-02).

## Fluxos alternativos

### FA-01 — Destinatário é o acompanhante no local

- **Condição:** a orientação é dada a um acompanhante ou ao solicitante, antes da chegada da equipe.
- **Passos:** 1. O regulador informa o nome do acompanhante e o vínculo com o paciente. 2. O sistema não solicita documento nem outro dado pessoal (minimização, LGPD).
- **Reconvergência:** retorna ao passo 4 do fluxo principal.

### FA-02 — Orientação sem despacho (desvio por falta de viatura)

- **Condição:** não há viatura disponível e o regulador orienta o solicitante por telefone.
- **Passos:** 1. O regulador seleciona um chamado classificado sem despacho ativo. 2. O sistema registra que a orientação é um desvio, sem equipe vinculada.
- **Reconvergência:** retorna ao passo 3 do fluxo principal.
- **Dependência aberta:** o contrato C2 não cobre esse caso. Ver questão Q3 no README do grupo.

### FA-03 — Orientação complementar ou retificação

- **Condição:** o regulador precisa complementar ou corrigir uma orientação já registrada.
- **Passos:** 1. O regulador cria uma nova teleorientação referenciando a anterior. 2. O sistema mantém a anterior intacta.
- **Reconvergência:** retorna ao passo 3 do fluxo principal.

## Fluxos de exceção

### FE-01 — Campo obrigatório ausente

- **Condição:** destinatário ou conteúdo não preenchido no passo 6.
- **Passos:** 1. O sistema bloqueia a gravação. 2. O sistema indica o campo faltante.
- **Estado final:** nada gravado, nenhum evento publicado.

### FE-02 — Chamado inexistente, cancelado ou encerrado

- **Condição:** o `chamado_id` selecionado não aceita novas orientações.
- **Passos:** 1. O sistema rejeita o registro e informa o estado do chamado. 2. O regulador volta à fila.
- **Estado final:** nada gravado.

### FE-03 — Auditoria (G6) indisponível

- **Condição:** o evento não pode ser entregue a G6 no passo 8.
- **Passos:** 1. O sistema conclui a gravação da teleorientação normalmente. 2. O evento fica pendente e é reentregue depois, com o mesmo `evento_id`.
- **Estado final:** teleorientação gravada, evento pendente. O atendimento não é bloqueado (ARQ §6).

## Regras de negócio aplicadas

| Regra | Descrição |
| --- | --- |
| RN-09 | passo 8: ação sensível gera evento imutável (autor, ação, entidade, horário) |
| RN-11 | passo 2: o regulador só vê dado clínico de chamado sob sua regulação |
| Res. CFM 2.314/2022, art. 13 | passo 7: nome e CRM do médico, data/hora e indicação de telemedicina (premissa a verificar, ver RNF-G03-05) |

## Relacionamentos

- Sem `<<include>>` nem `<<extend>>`. O caso de uso é autônomo: a orientação é uma ação independente do preenchimento da ficha.
- Relacionado a UC-G03-02: a orientação registrada é exibida na ficha.

## Dados manipulados

| Entidade | Dado | Origem |
| --- | --- | --- |
| Chamado | `chamado_id` | referência a G1 |
| Despacho | `despacho_id`, `equipe_id` | contrato C2 (G2) |
| Teleorientacao | autor, horário do fato, destinatário, conteúdo | este módulo |

## Critérios de aceite

- [ ] Toda teleorientação gravada tem autor, horário do fato, destinatário e conteúdo.
- [ ] Teleorientação anterior nunca é alterada; correção cria novo registro.
- [ ] Cada teleorientação gera exatamente um evento `TELEORIENTACAO_REGISTRADA`.
- [ ] Falha de G6 não impede o registro nem o atendimento.

## Rastreabilidade

| RF | Classe | Sequência | Teste |
| --- | --- | --- | --- |
| RF-G03-01 | pendente (M2) | pendente (M2) | pendente (M3) |
| RF-G03-02 | pendente (M2) | pendente (M2) | pendente (M3) |
