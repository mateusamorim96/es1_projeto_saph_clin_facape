# Modelagem Comportamental — G03 Telemedicina e Prontuário (E5 · v1.0)

> Índice dos diagramas de comportamento e das regras de transição. Os diagramas estão em
> [`diagramas/`](diagramas/) (fonte `.puml` e exportações `.svg` e `.png`). Os estados, atributos e
> mensagens usam os nomes do [`dicionario-de-dados.md`](dicionario-de-dados.md).

## 1. Diagramas de sequência (6)

O E5 exige ao menos 2, com pelo menos um de exceção ou de indisponibilidade. Aqui, quatro dos seis tratam
exceção ou indisponibilidade de forma explícita (marcados em **negrito**).

| ID | Diagrama | Caso de uso | Requisitos | O que mostra |
| --- | --- | --- | --- | --- |
| DS-G03-01 | [`ds-g03-01-registrar-teleorientacao`](diagramas/ds-g03-01-registrar-teleorientacao.puml) | UC-G03-01 | RF-G03-01, 02 | Registro da orientação e publicação do evento. **Exceções:** campo ausente (FE-01), chamado encerrado (FE-02) e **G6 indisponível (FE-03)**, com evento pendente e reentrega |
| DS-G03-02 | [`ds-g03-02-solicitar-pre-admissao`](diagramas/ds-g03-02-solicitar-pre-admissao.puml) | UC-G03-03, 05, 07 | RF-G03-09, 10, 11, 15 | Fluxo crítico principal: validação (`<<include>>`), envio do C3, reserva em G4 e consulta pela recepção |
| DS-G03-03 | [`ds-g03-03-pre-admissao-indisponibilidade`](diagramas/ds-g03-03-pre-admissao-indisponibilidade.puml) | UC-G03-03 | RF-G03-11 | **Indisponibilidade:** sem conexão (FE-03), leito negado (FE-01) e G4 sem resposta (FE-04), com reenvio pelo mesmo `evento_id` |
| DS-G03-04 | [`ds-g03-04-ficha-offline-sincronizacao`](diagramas/ds-g03-04-ficha-offline-sincronizacao.puml) | UC-G03-02, 06, 08 | RF-G03-03 a 08 | Abertura da ficha por C2, registro offline, C4 de fármaco e sincronização idempotente. **Exceção:** falha de gravação local e **nova queda durante a sincronização** |
| DS-G03-05 | [`ds-g03-05-registrar-recusa`](diagramas/ds-g03-05-registrar-recusa.puml) | UC-G03-04 | RF-G03-12, 13 | Recusa com assinatura, testemunha e ciência. **Exceções:** consciência não confirmada, paciente não assina, evidência incompleta |
| DS-G03-06 | [`ds-g03-06-consultar-ficha-acesso`](diagramas/ds-g03-06-consultar-ficha-acesso.puml) | UC-G03-07 | RF-G03-14, 15 | Acesso permitido e acesso negado, ambos registrados como evento |

## 2. Máquinas de estado (3)

O ciclo de vida central do módulo é o do `Prontuario` (ME-G03-01). As outras duas entidades com ciclo
próprio têm máquina separada.

| ID | Diagrama | Entidade |
| --- | --- | --- |
| ME-G03-01 | [`me-g03-01-estados-prontuario`](diagramas/me-g03-01-estados-prontuario.puml) | `Prontuario` (ciclo central) |
| ME-G03-02 | [`me-g03-02-estados-pre-admissao`](diagramas/me-g03-02-estados-pre-admissao.puml) | `PreAdmissao` |
| ME-G03-03 | [`me-g03-03-estados-recusa-atendimento`](diagramas/me-g03-03-estados-recusa-atendimento.puml) | `RecusaAtendimento` |

### 2.1 Transições do Prontuario (ME-G03-01)

| Origem | Evento | Guarda | Destino | Requisito |
| --- | --- | --- | --- | --- |
| (início) | C2 `despacho_ativo` recebido | `despacho_id` ainda sem ficha | `ABERTO` | RF-G03-03 |
| `ABERTO` | primeiro registro clínico gravado | nenhuma | `EM_PREENCHIMENTO` | RF-G03-04 a 06 |
| `ABERTO` ou `EM_PREENCHIMENTO` | recusa assinada | recusa em `ASSINADA` | `RECUSADO` | RF-G03-12 |
| `EM_PREENCHIMENTO` | C3 publicado | `PreAdmissao` em `ENVIADA` | `AGUARDANDO_LEITO` | RF-G03-09 |
| `AGUARDANDO_LEITO` | G4 reservou o leito | resposta `reservado` | `LEITO_CONFIRMADO` | RF-G03-11 |
| `AGUARDANDO_LEITO` | G4 negou ou não respondeu | `NEGADA` ou `SEM_RESPOSTA` | `EM_PREENCHIMENTO` | RF-G03-11 |
| `AGUARDANDO_LEITO` | necessidade mudou | novo pedido enviado | `AGUARDANDO_LEITO` | RF-G03-09 |
| `LEITO_CONFIRMADO` | deterioração clínica | novo pedido enviado | `AGUARDANDO_LEITO` | RF-G03-09 |
| `RECUSADO` | recusa revogada | recusa em `REVOGADA` | `EM_PREENCHIMENTO` | RF-G03-12 |
| `EM_PREENCHIMENTO`, `LEITO_CONFIRMADO` ou `RECUSADO` | despacho encerrado | nenhum registro `PENDENTE` | `ENCERRADO` | RF-G03-08 |

Proibido: enviar pré-admissão em `RECUSADO`; sair de `ENCERRADO`; editar `prioridade_herdada` em qualquer estado.

### 2.2 Transições da PreAdmissao (ME-G03-02)

| Origem | Evento | Guarda | Destino | Requisito |
| --- | --- | --- | --- | --- |
| (início) | regulador aciona o pedido | ficha não está `RECUSADO` | `RASCUNHO` | RF-G03-09 |
| `RASCUNHO` | validação falha | campo faltante | `RASCUNHO` | RF-G03-10 |
| `RASCUNHO` | confirmar envio | validação aprovada, com conexão | `ENVIADA` | RF-G03-09 |
| `RASCUNHO` | confirmar envio | validação aprovada, sem conexão | `PENDENTE_DE_ENVIO` | RF-G03-09 |
| `PENDENTE_DE_ENVIO` | conexão restabelecida | mesmo `evento_id` | `ENVIADA` | RF-G03-08 |
| `ENVIADA` | G4 reservou | resposta `reservado` | `CONFIRMADA` | RF-G03-11 |
| `ENVIADA` | G4 negou | resposta `negado` com motivo | `NEGADA` | RF-G03-11 |
| `ENVIADA` | prazo esgotado | sem resposta (premissa: 60 s) | `SEM_RESPOSTA` | RF-G03-11 |
| `ENVIADA` ou `CONFIRMADA` | novo pedido por mudança de necessidade | novo pedido enviado | `SUBSTITUIDA` | RF-G03-09 |
| `SEM_RESPOSTA` | reenvio manual | nenhuma | `ENVIADA` | RF-G03-11 |
| `SEM_RESPOSTA` | resposta tardia | `reservado` ou `negado` | `CONFIRMADA` ou `NEGADA` | RF-G03-11 |

### 2.3 Transições da RecusaAtendimento (ME-G03-03)

| Origem | Evento | Guarda | Destino | Requisito |
| --- | --- | --- | --- | --- |
| (início) | recusa iniciada | consciência confirmada | `RASCUNHO` | RF-G03-12 |
| `RASCUNHO` | evidência faltante | nenhuma | `RASCUNHO` | RF-G03-12 |
| `RASCUNHO` | três evidências presentes | assinatura, testemunha e ciência (RN-10) | `ASSINADA` | RF-G03-12, 13 |
| `RASCUNHO` | paciente não quer assinar | testemunha e ciência presentes | `NAO_ASSINADA` | RF-G03-12 |
| `ASSINADA` ou `NAO_ASSINADA` | paciente aceita o atendimento | nenhuma | `REVOGADA` | RF-G03-12 |

## 3. Diagramas de atividades (2)

O módulo tem decisões de fluxo em dois pontos. Cada diagrama usa raias para separar quem decide.

| ID | Diagrama | Decisões modeladas |
| --- | --- | --- |
| DA-G03-01 | [`da-g03-01-solicitar-pre-admissao`](diagramas/da-g03-01-solicitar-pre-admissao.puml) | Sinais vitais existem? Prioridade vigente existe? Necessidade e ETA preenchidos? Há conexão? Resposta de G4: reservado, negado ou sem resposta |
| DA-G03-02 | [`da-g03-02-registrar-recusa`](diagramas/da-g03-02-registrar-recusa.puml) | Consciência confirmada? Paciente é menor ou incapaz? Assinatura obtida? Três evidências presentes? Paciente muda de ideia? |

## 4. Coerência entre os diagramas

- As mensagens das sequências usam os contratos C2, C3, C4 e C5 e os mesmos campos do dicionário de dados.
- Os estados que aparecem nas sequências (`PENDENTE_DE_ENVIO`, `ENVIADA`, `NEGADA`, `SEM_RESPOSTA`, `ASSINADA`...) existem nas máquinas de estado e na enumeração do dicionário.
- A prioridade nunca é calculada nem editada em nenhum diagrama (RN-02, ADR-G03-01).
- Eventos de G6 saem sempre do G3 para G6 e não bloqueiam o fluxo clínico quando G6 está indisponível.
- Decisões que mudam contrato (ADR-G03-01, 04 e 05) estão marcadas como `proposto`, porque dependem de outro grupo.
