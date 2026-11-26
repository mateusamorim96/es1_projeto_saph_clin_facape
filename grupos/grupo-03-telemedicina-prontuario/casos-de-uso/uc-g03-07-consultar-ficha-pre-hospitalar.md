# UC-G03-07 — Consultar Ficha Pré-Hospitalar

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | UC-G03-07 |
| **Módulo** | G03 — Telemedicina e Prontuário Pré-Hospitalar |
| **Ator primário** | EquipeReceptora (leitura); MedicoRegulador e EquipeIntervencao também consultam |
| **Atores secundários** | Módulo G6 (recebe o evento de acesso) |
| **Objetivo** | Permitir a leitura da ficha com controle de perfil e registro de cada acesso (passagem de plantão digital) |
| **Requisitos rastreados** | RF-G03-14, RF-G03-15 |
| **Prioridade** | essencial |

## Pré-condições

- Usuário autenticado.
- Para a `EquipeReceptora`, existe pré-admissão enviada para o atendimento.

## Pós-condições

- **Sucesso:** ficha exibida em somente leitura; acesso registrado e evento `PRONTUARIO_ACESSADO` publicado.
- **Falha:** nenhum conteúdo exibido; tentativa registrada com resultado `NEGADO`.

## Fluxo principal

1. O usuário seleciona o atendimento na lista de pacientes a caminho ou na central de regulação.
2. O sistema verifica o perfil e a necessidade de atendimento (tabela abaixo; RN-11).
3. O sistema registra o acesso com ator, papel, horário e resultado (RF-G03-15).
4. O sistema publica `PRONTUARIO_ACESSADO` pelo C5 e exibe a ficha em leitura: sinais vitais (histórico e último), procedimentos, fármacos, teleorientações, recusa e pedido com desfecho.

## Perfis com necessidade de atendimento (RN-11 / P-06)

| Perfil | Acesso |
| --- | --- |
| MedicoRegulador | atendimentos que regula |
| EquipeIntervencao | ficha do próprio `equipe_id` |
| EquipeReceptora | atendimentos com pré-admissão enviada |
| Auditor | sem conteúdo clínico; consulta apenas a trilha de acessos em G06 |

## Fluxos alternativos

### FA-01 — Dados possivelmente defasados

- **Condição:** a equipe de campo pode estar sem conexão.
- **Passos:** 1. O sistema exibe o horário do último registro recebido. 2. O usuário considera a defasagem.
- **Reconvergência:** fim do fluxo.

### FA-02 — Acesso offline no tablet

- **Condição:** sem conexão com a central.
- **Passos:** 1. O registro de acesso é gravado localmente. 2. UC-G03-08 o sincroniza na reconexão.
- **Reconvergência:** retorna ao passo 4.

## Fluxos de exceção

### FE-01 — Perfil sem necessidade de atendimento

- **Condição:** o usuário não tem papel ou vínculo com o atendimento.
- **Passos:** 1. O sistema nega o acesso, sem exibir conteúdo. 2. Registra a tentativa com resultado `NEGADO`.
- **Estado final:** nada exibido; evento `PRONTUARIO_ACESSADO` com resultado `NEGADO`.

### FE-02 — Falha na publicação do evento

- **Condição:** G6 indisponível.
- **Passos:** 1. O acesso permitido continua, por continuidade do atendimento. 2. O evento fica pendente e é reentregue com o mesmo `evento_id`.
- **Estado final:** acesso concedido; evento pendente.

## Regras de negócio aplicadas

| Regra | Aplicação |
| --- | --- |
| RN-11 | passo 2: dado clínico só para perfil com necessidade de atendimento |
| RN-09 | passo 4: o acesso gera evento imutável |
| LGPD art. 37 | registro das operações de tratamento de dados |

## Relacionamentos

- Autônomo, conforme o catálogo (`uc-g03-00`). A verificação de perfil e o evento de acesso fazem parte do próprio fluxo (RF-G03-14 e RF-G03-15).

## Dados manipulados

| Entidade | Dado | Origem |
| --- | --- | --- |
| Prontuario e registros | conteúdo da ficha (somente leitura) | este módulo |
| PreAdmissao | pedido e desfecho | este módulo |
| PublicacaoEvento | `evento_id`, tipo, ator, papel, resultado | este módulo (C5 para G6) |

## Critérios de aceite

- [ ] A consulta é somente leitura.
- [ ] Perfil sem necessidade (por exemplo, Auditor) não vê conteúdo clínico.
- [ ] Cada acesso, permitido ou negado, gera evento com `evento_id` único.
- [ ] Falha de G06 não bloqueia o atendimento e não perde o evento.

## Rastreabilidade

| RF | Classe | Sequência | Teste |
| --- | --- | --- | --- |
| RF-G03-14 | Prontuario | DS-G03-06 | CT-G03-06 |
| RF-G03-15 | Prontuario, PublicacaoEvento | DS-G03-06 | CT-G03-06 |
