# Dicionário de Dados — G03 Telemedicina e Prontuário (E4 · v1.0)

> Acompanha o diagrama de classes [`diagramas/diagrama-classes-g03.puml`](diagramas/diagrama-classes-g03.puml)
> (exportações `.svg` e `.png` na mesma pasta). Para cada classe: significado, atributos relevantes,
> restrições e relacionamentos. Tipos `Id…` são identificadores opacos.
> Tipos marcados «G1», «G2», «G4», «G5», «G6» são **referências por identificador** a entidades de outros módulos.
> O módulo não as modela como classes (regra de coerência 1).

## 1. Visão das classes

| Classe | Papel | Dono | RF principais |
| --- | --- | --- | --- |
| `Prontuario` | Ficha pré-hospitalar de um despacho; raiz do agregado | G3 | RF-G03-03, 14, 15 |
| `RegistroClinico` (abstrata) | Registro clínico imutável e versionado, feito em campo | G3 | RF-G03-04, 07, 08 |
| `RegistroSinaisVitais` | Medição de sinais vitais em um horário | G3 | RF-G03-04 |
| `Procedimento` | Procedimento realizado no atendimento | G3 | RF-G03-05 |
| `ConsumoFarmaco` | Fármaco administrado, origem do contrato C4 | G3 | RF-G03-06 |
| `Teleorientacao` | Orientação do médico regulador antes da chegada | G3 | RF-G03-01, 02 |
| `PreAdmissao` | Pedido de leito enviado a G4, origem do contrato C3 | G3 | RF-G03-09, 10, 11 |
| `RecusaAtendimento` | Recusa de atendimento por paciente consciente | G3 | RF-G03-12, 13 |
| `PublicacaoEvento` | Fila de entrega de eventos sensíveis (C5) a G6 | G3 | RF-G03-02, 13, 15 |

Herança: `RegistroSinaisVitais`, `Procedimento` e `ConsumoFarmaco` herdam de `RegistroClinico`
(compartilham autoria, horários do fato e do registro, estado de sincronização e versionamento).

## 2. Classes

### 2.1 Prontuario

**Significado.** Ficha pré-hospitalar digital de um atendimento. É criada quando o contrato C2 chega e
agrupa tudo o que foi registrado em campo. Não é o prontuário hospitalar definitivo (fora do escopo).

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `prontuario_id` | IdProntuario | sim | identificador único |
| `chamado_id` | IdChamado «G1» | sim | referência; o chamado pertence a G1 |
| `despacho_id` | IdDespacho «G2» | sim | **único**: um despacho tem no máximo uma ficha (RF-G03-03) |
| `equipe_id` | IdEquipe «G2» | sim | base do controle de acesso da `EquipeIntervencao` (RF-G03-14) |
| `prioridade_herdada` | Prioridade | sim | **somente leitura**; vem do chamado (ADR-G03-01), nunca é recalculada |
| `prioridade_atualizada_em` | DataHora | sim | horário em que a prioridade vigente foi recebida |
| `estado` | EstadoProntuario | sim | ciclo de vida em ME-G03-01 |
| `aberto_em` | DataHora | sim | horário de abertura |
| `encerrado_em` | DataHora | não | só preenchido em `ENCERRADO`; exige 0 registros `PENDENTE` |

**Relacionamentos.** Compõe 0..* `RegistroClinico`, 0..* `PreAdmissao` e 0..* `RecusaAtendimento`.
Agrega 0..* `Teleorientacao` (ligação opcional: a teleorientação pode existir antes da ficha).

**Invariantes.** (1) Um `despacho_id` não gera duas fichas. (2) Pré-admissão não pode ser enviada em
estado `RECUSADO`. (3) `prioridade_herdada` não é editável por nenhum caso de uso deste módulo (RN-02).

### 2.2 RegistroClinico (abstrata)

**Significado.** Registro feito pela equipe em campo. Nunca é alterado nem apagado: uma correção cria
nova versão e a original permanece (ADR-G03-02).

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `registro_id` | IdRegistro | sim | identificador único, também usado como chave de idempotência |
| `responsavel_id` | IdUsuario | sim | quem registrou ou executou |
| `ocorrido_em` | DataHora | sim | horário do **fato** (coleta, realização, administração); não é o horário de digitação |
| `registrado_em` | DataHora | sim | horário em que o registro foi gravado |
| `sincronizacao` | EstadoSincronizacao | sim | `PENDENTE` até a central confirmar (ADR-G03-03) |
| `versao` | Inteiro | sim | ≥ 1; a versão 1 é o registro original |
| `substitui_id` | IdRegistro | não | obrigatório se `versao` > 1; aponta a versão anterior |
| `motivo_correcao` | Texto | condicional | obrigatório se `versao` > 1 |
| `sinalizado_para_revisao` | Booleano | sim | verdadeiro para registro com `ocorrido_em` posterior ao encerramento do despacho |

**Restrições.** `ocorrido_em` ≤ `registrado_em`. Uma versão só é substituída uma vez (cadeia linear).

### 2.3 RegistroSinaisVitais (herda de RegistroClinico)

**Significado.** Conjunto de medições feitas no mesmo horário.

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `frequencia_cardiaca_bpm` | Inteiro | não | unidade fixa em bpm |
| `pressao_sistolica_mmhg`, `pressao_diastolica_mmhg` | Inteiro | não | em mmHg; informadas juntas |
| `frequencia_respiratoria_irpm` | Inteiro | não | em incursões por minuto |
| `saturacao_o2_pct` | Inteiro | não | em % |
| `temperatura_c` | Decimal | não | em °C |
| `glicemia_mgdl` | Inteiro | não | em mg/dL |
| `escala_glasgow` | Inteiro | não | escala de coma de Glasgow |

**Restrição.** Pelo menos um valor preenchido. Valor implausível pede confirmação, mas não impede o
registro (a faixa plausível é definida no M3; a decisão clínica é da equipe). O último registro vigente
alimenta o `sinais_vitais` do C3.

### 2.4 Procedimento (herda de RegistroClinico)

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `tipo` | Texto | sim | catálogo de procedimentos a fechar no M3 |
| `descricao` | Texto | não | detalhe livre |

O horário do procedimento é `ocorrido_em`; o responsável é `responsavel_id`.

### 2.5 ConsumoFarmaco (herda de RegistroClinico)

**Significado.** Fármaco administrado ao paciente. Cada registro gera um contrato C4 para G5.
A baixa de estoque e a validade do lote são de G5.

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `item_id` | IdItem «G5» | sim | item da maleta; entidade de G5 |
| `lote` | Texto | sim | sem lote, a gravação é bloqueada (RN-07) |
| `quantidade` | Decimal | sim | > 0 |
| `unidade` | Texto | sim | unidade de medida do item |
| `via` | ViaAdministracao | sim | catálogo fechado |
| `evento_id` | IdEvento | sim | chave de idempotência do C4 |

O horário de administração é `ocorrido_em` (campo `horario` do C4); o `responsavel_id` do C4 é o da base.

### 2.6 Teleorientacao

**Significado.** Orientação dada pelo médico regulador a quem está no local, antes da chegada da viatura.
É um ato de telemedicina e carrega a identificação exigida pela norma (RNF-G03-05).

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `teleorientacao_id` | IdTeleorientacao | sim | identificador único |
| `chamado_id` | IdChamado «G1» | sim | vínculo principal |
| `despacho_id` | IdDespacho «G2» | não | vazio no desvio por falta de viatura (ADR-G03-05) |
| `medico_id`, `medico_nome`, `medico_crm` | IdUsuario, Texto, Texto | sim | identificação do médico no documento (Res. CFM 2.314/2022, art. 13) |
| `destinatario_tipo` | DestinatarioTipo | sim | `EQUIPE`, `ACOMPANHANTE` ou `SOLICITANTE` |
| `destinatario_nome`, `destinatario_vinculo` | Texto | condicional | obrigatórios se o tipo não for `EQUIPE`; só nome e vínculo (minimização, LGPD) |
| `conteudo` | Texto | sim | a orientação dada |
| `ocorrida_em` | DataHora | sim | horário do fato |
| `registrada_em` | DataHora | sim | horário da gravação |
| `emitida_por_telemedicina` | Booleano | sim | sempre verdadeiro; indicação exigida na norma |
| `desvio_sem_viatura` | Booleano | sim | verdadeiro quando não há despacho |
| `retifica_id` | IdTeleorientacao | não | aponta a orientação complementada ou corrigida |

**Relacionamentos.** Pode ser exibida por 0..1 `Prontuario`. Retifica 0..1 `Teleorientacao`.
**Restrição.** Imutável após gravada; a correção cria novo registro (FA-03 do UC-G03-01).

### 2.7 PreAdmissao

**Significado.** Pedido antecipado de leito enviado a G4. É a origem do contrato C3. Esta classe guarda o
**pedido** e o resultado devolvido por G4; a reserva do leito pertence a G4.

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `pre_admissao_id` | IdPreAdmissao | sim | identificador único |
| `prioridade` | Prioridade | sim | **cópia** da prioridade herdada no momento do envio; imutável |
| `necessidade` | Necessidade | sim | `SALA_VERMELHA` ou `OBSERVACAO` |
| `eta_hospital` | Duracao | sim | tempo estimado até o hospital |
| `estado` | EstadoPreAdmissao | sim | ciclo de vida em ME-G03-02 |
| `motivo_negativa` | Texto | condicional | obrigatório em `NEGADA` |
| `solicitada_por` | IdUsuario | sim | médico regulador |
| `criada_em`, `enviada_em`, `respondida_em` | DataHora | parcial | `enviada_em` e `respondida_em` só após o envio e a resposta |
| `evento_id` | IdEvento | não | gerado no envio; mesma chave em reenvios (idempotência) |
| `reserva_ref` | IdReserva «G4» | não | referência devolvida por G4 em `CONFIRMADA` |
| `substitui_id` | IdPreAdmissao | não | pedido anterior substituído por mudança de necessidade |

**Relacionamentos.** Pertence a 1 `Prontuario`. Referencia 1 `RegistroSinaisVitais` (o último registro enviado).
Substitui 0..1 `PreAdmissao`.
**Invariantes.** Só existe um pedido ativo (`ENVIADA`, `CONFIRMADA`, `PENDENTE_DE_ENVIO` ou `SEM_RESPOSTA`) por ficha.
Não é enviada sem ao menos um `RegistroSinaisVitais`.

### 2.8 RecusaAtendimento

**Significado.** Registro com evidência da recusa de atendimento ou transporte por paciente consciente (RN-10).

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `recusa_id` | IdRecusa | sim | identificador único |
| `estado` | EstadoRecusa | sim | ciclo de vida em ME-G03-03 |
| `consciencia_confirmada` | Booleano | sim | deve ser verdadeiro, senão a recusa não é criada |
| `riscos_informados`, `motivo` | Texto | sim | o que foi dito ao paciente e a razão da recusa |
| `assinante_tipo` | AssinanteTipo | sim | `PACIENTE` ou `REPRESENTANTE_LEGAL` |
| `representante_vinculo` | Texto | condicional | obrigatório se o assinante for representante |
| `testemunha_nome`, `testemunha_vinculo` | Texto | sim | identificação mínima da testemunha |
| `assinatura_paciente_id`, `assinatura_testemunha_id`, `ciencia_regulador_id` | IdAssinatura «G6» | condicional | as três são obrigatórias em `ASSINADA`; a assinatura é entidade de G6 |
| `motivo_sem_assinatura` | Texto | condicional | obrigatório em `NAO_ASSINADA` |
| `registrada_em`, `revogada_em` | DataHora | parcial | `revogada_em` só em `REVOGADA` |
| `motivo_revogacao` | Texto | condicional | obrigatório em `REVOGADA` |

**Invariantes.** Só `ASSINADA` satisfaz a RN-10 e publica `RECUSA_ASSINADA`. A revogação não apaga a recusa original.

### 2.9 PublicacaoEvento

**Significado.** Controle de entrega de um evento sensível a G6 (contrato C5). É uma **fila de entrega**,
não uma trilha de auditoria: guarda só o necessário para reentregar sem duplicar. A trilha imutável é de G6.

| Atributo | Tipo | Obrig. | Restrição |
| --- | --- | --- | --- |
| `evento_id` | IdEvento | sim | identificador imutável; reenvio não duplica |
| `tipo` | TipoEvento | sim | `TELEORIENTACAO_REGISTRADA`, `RECUSA_ASSINADA` ou `PRONTUARIO_ACESSADO` |
| `ator_id`, `papel` | IdUsuario, Papel | sim | quem executou e com que perfil |
| `entidade`, `entidade_id` | Texto, Identificador | sim | o que sofreu a ação; só o identificador, nunca o conteúdo clínico |
| `ocorrido_em` | DataHora | sim | horário do fato |
| `resultado` | ResultadoEvento | sim | `SUCESSO`, `NEGADO` ou `ERRO` |
| `justificativa` | Texto | condicional | obrigatória na recusa (o motivo) |
| `estado_entrega` | EstadoEntrega | sim | `PENDENTE` até G6 confirmar |
| `tentativas` | Inteiro | sim | ≥ 0 |

**Restrição de minimização (LGPD).** O evento não carrega dado clínico, só o identificador da entidade.

## 3. Tipos enumerados

| Enumeração | Valores | Observação |
| --- | --- | --- |
| `Prioridade` | `VERMELHO`, `LARANJA`, `AMARELO`, `VERDE`, `AZUL` | valores do contrato C1; definidos por G1 |
| `Necessidade` | `SALA_VERMELHA`, `OBSERVACAO` | valores do contrato C3 |
| `EstadoProntuario` | `ABERTO`, `EM_PREENCHIMENTO`, `AGUARDANDO_LEITO`, `LEITO_CONFIRMADO`, `RECUSADO`, `ENCERRADO` | ME-G03-01 |
| `EstadoPreAdmissao` | `RASCUNHO`, `PENDENTE_DE_ENVIO`, `ENVIADA`, `CONFIRMADA`, `NEGADA`, `SEM_RESPOSTA`, `SUBSTITUIDA` | ME-G03-02 |
| `EstadoRecusa` | `RASCUNHO`, `ASSINADA`, `NAO_ASSINADA`, `REVOGADA` | ME-G03-03 |
| `EstadoSincronizacao` | `PENDENTE`, `SINCRONIZADO` | ADR-G03-03 |
| `EstadoEntrega` | `PENDENTE`, `ENTREGUE` | entrega de eventos a G6 |
| `TipoEvento` | `TELEORIENTACAO_REGISTRADA`, `RECUSA_ASSINADA`, `PRONTUARIO_ACESSADO` | catálogo de eventos sensíveis de G3 |
| `ResultadoEvento` | `SUCESSO`, `NEGADO`, `ERRO` | contrato C5 |
| `Papel` | `MEDICO_REGULADOR`, `EQUIPE_INTERVENCAO`, `EQUIPE_RECEPTORA` | nomes do glossário |
| `DestinatarioTipo` | `EQUIPE`, `ACOMPANHANTE`, `SOLICITANTE` | |
| `AssinanteTipo` | `PACIENTE`, `REPRESENTANTE_LEGAL` | |
| `ViaAdministracao` | `INTRAVENOSA`, `INTRAMUSCULAR`, `SUBCUTANEA`, `ORAL`, `SUBLINGUAL`, `INALATORIA`, `OUTRA` | catálogo a fechar com G5 no M3 |

## 4. Referências a outros módulos (não modeladas aqui)

| Identificador | Dono | Usado em | Contrato |
| --- | --- | --- | --- |
| `chamado_id` | G1 | `Prontuario`, `Teleorientacao` | C1, C2 |
| `despacho_id`, `equipe_id` | G2 | `Prontuario`, `Teleorientacao` | C2 |
| `item_id` | G5 | `ConsumoFarmaco` | C4 |
| `reserva_ref` | G4 | `PreAdmissao` | resposta ao C3 |
| `assinatura_*_id` | G6 | `RecusaAtendimento` | contrato novo proposto no ADR-G03-04 |

## 5. Mapa de contratos para as classes

| Contrato | Direção | Classe de origem ou destino | Campos |
| --- | --- | --- | --- |
| C2 `despacho_ativo` | G2 → G3 | `Prontuario` | `despacho_id`, `chamado_id`, `equipe_id` (+ `prioridade`, se o ADR-G03-01 for aceito) |
| C3 `pedido_pre_admissao` | G3 → G4 | `PreAdmissao`, `Prontuario`, `RegistroSinaisVitais` | `prontuario_id`, `chamado_id`, `prioridade`, `necessidade`, `sinais_vitais`, `eta_hospital` |
| C4 `consumo_farmaco` | G3 → G5 | `ConsumoFarmaco` | `item_id`, `lote`, `quantidade`, `via`, `horario`, `responsavel_id`, `prontuario_id` |
| C5 `evento_sensivel` | G3 → G6 | `PublicacaoEvento` | `evento_id`, `tipo`, `ator_id`/`papel`, `entidade`/`entidade_id`, `ocorrido_em`, `resultado`, `justificativa` |
