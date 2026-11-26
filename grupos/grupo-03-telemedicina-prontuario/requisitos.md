# Catálogo de Requisitos — G03 Telemedicina e Prontuário (E2 · v1.0)

> Marco M1. Módulo M2 · Telemedicina Regulada e Prontuário Integrado.
> Entidades de outros grupos aparecem só por referência (`chamado_id`, `despacho_id`).
> Prioridade clínica é do G1 e auditoria é do G6: este módulo herda e publica, não recalcula nem duplica.

## 1. Requisitos Funcionais

Atores seguem o glossário: `MedicoRegulador`, `EquipeIntervencao`, `EquipeReceptora`.
"Sistema" indica ação automática do módulo. Fontes: `DEF` = `definicao-projeto.md`,
`CTX` = `contexto-do-sistema.md`, `ARQ` = `arquitetura-modular.md`, `TEM` = `temas-grupos.md`.

| ID | Ator | Requisito | Critério de aceite | Fonte | UC |
| --- | --- | --- | --- | --- | --- |
| RF-G03-01 | MedicoRegulador | Registrar teleorientação com autor, horário do fato, destinatário e conteúdo da orientação, vinculada a um `chamado_id`. | Dado um chamado válido, ao confirmar, existe uma `Teleorientacao` com os quatro campos preenchidos. Sem destinatário ou sem conteúdo, nada é gravado. | DEF (Módulo 2), TEM (G3) | UC-G03-01 |
| RF-G03-02 | Sistema | Publicar o evento `TELEORIENTACAO_REGISTRADA` ao gravar uma teleorientação. | Cada teleorientação gravada gera exatamente um evento com `evento_id`, `ator_id`, `entidade_id` e `ocorrido_em` iguais aos do registro. Reenvio do mesmo `evento_id` não duplica. | ARQ (C5, §4, §6), RN-09 | UC-G03-01 |
| RF-G03-03 | Sistema | Abrir a ficha pré-hospitalar ao receber o contrato C2 `despacho_ativo`, vinculada a `chamado_id` e `despacho_id`. | Um C2 gera exatamente uma ficha aberta. Um segundo C2 com o mesmo `despacho_id` não cria outra ficha. | ARQ (C2) | UC-G03-02 |
| RF-G03-04 | EquipeIntervencao | Registrar sinais vitais em campo, cada registro com horário de coleta e responsável, mantendo o histórico. | Dois registros em horários diferentes coexistem na ficha. Uma correção cria nova versão e preserva o valor original, o autor e o motivo. | DEF (Módulo 2), TEM (G3) | UC-G03-02 |
| RF-G03-05 | EquipeIntervencao | Registrar procedimentos realizados no atendimento, com tipo, horário e responsável. | Cada procedimento aparece na ficha com os três campos. Procedimento sem horário não é gravado. | DEF (Módulo 2) | UC-G03-02 |
| RF-G03-06 | EquipeIntervencao | Registrar fármaco administrado (item, lote, quantidade, via, horário, responsável) e publicar o contrato C4 `consumo_farmaco`. | Sem lote, a gravação é bloqueada. Ao gravar, o C4 sai com os sete campos obrigatórios, incluindo `prontuario_id`. A baixa de estoque é de G5, não deste módulo. | ARQ (C4), RN-07 | UC-G03-06 |
| RF-G03-07 | EquipeIntervencao | Manter a ficha disponível para leitura e edição sem conexão, guardando localmente cada registro. | Com a conexão desligada, é possível abrir a ficha, incluir e corrigir registros. Cada registro fica com o estado "pendente de sincronização". | TEM (G3), seed do professor | UC-G03-02, UC-G03-08 |
| RF-G03-08 | Sistema | Sincronizar registros pendentes após a reconexão sem duplicar nem perder nenhum. | Reenviar o mesmo lote de registros três vezes resulta em uma única cópia de cada. A ordem segue o horário do fato, não o do envio. | ARQ (§6) | UC-G03-08 |
| RF-G03-09 | MedicoRegulador | Emitir o pedido de pré-admissão (C3) com `prontuario_id`, `chamado_id`, `prioridade`, `necessidade`, último `sinais_vitais` e `eta_hospital`. | O C3 sai com os seis campos. `necessidade` só aceita `SALA_VERMELHA` ou `OBSERVACAO`. Sem sinais vitais registrados, o pedido não é emitido. | ARQ (C3), DEF (Módulo 2) | UC-G03-03 |
| RF-G03-10 | Sistema | Validar, antes de emitir o C3, que a prioridade foi herdada do chamado e que os campos obrigatórios estão completos. | A prioridade é somente leitura neste módulo. Se faltar `prioridade`, sinais vitais ou `eta_hospital`, o envio é bloqueado e o campo faltante é indicado. | ARQ (C3), regra 4 de coerência, RN-02 | UC-G03-05 |
| RF-G03-11 | Sistema | Tratar a resposta de G4 ao pedido: registrar "confirmada" ou "negada" com motivo e avisar o médico regulador. | Resposta `negado` deixa a pré-admissão em "negada" com o motivo visível e libera o regulador para decidir o desvio. Nenhuma prioridade é recalculada. | ARQ (C3, falha de contrato) | UC-G03-03 |
| RF-G03-12 | EquipeIntervencao | Registrar recusa de atendimento de paciente consciente com assinatura do paciente (ou representante), testemunha identificada e ciência do médico regulador. | Falta de qualquer uma das três evidências mantém a recusa como rascunho e impede o evento. Paciente sem consciência confirmada não permite o registro. | RN-10, DEF (Módulo 4) | UC-G03-04 |
| RF-G03-13 | Sistema | Publicar o evento `RECUSA_ASSINADA` quando a recusa estiver completa. | O evento sai com `justificativa` (motivo da recusa) preenchida. Recusa em rascunho não gera evento. | ARQ (C5, §4) | UC-G03-04 |
| RF-G03-14 | Sistema | Restringir o acesso à ficha por perfil, conforme a necessidade de atendimento. | `EquipeIntervencao` acessa só fichas do seu `equipe_id`. `EquipeReceptora` acessa só fichas com pré-admissão enviada. Outros perfis recebem acesso negado. | RN-11, LGPD art. 5º, II | UC-G03-07 |
| RF-G03-15 | Sistema | Registrar cada acesso à ficha, permitido ou negado, e publicar o evento `PRONTUARIO_ACESSADO`. | Cada abertura de ficha gera um evento com perfil, ator, `prontuario_id` e `resultado` (`SUCESSO` ou `NEGADO`). | ARQ (C5, §4), RN-09, RN-11 | UC-G03-07 |

### Rastro das sementes do README

O README do grupo trazia 8 RF-semente e 5 RNF-semente. Eles foram analisados, divididos
quando eram compostos e renumerados. A tabela evita perder a ligação com a versão anterior.

| Semente | Texto resumido | Virou |
| --- | --- | --- |
| RF-G03-01 | teleorientação com autor, horário, destinatário | RF-G03-01 |
| RF-G03-02 | sinais vitais com horário de coleta | RF-G03-04 |
| RF-G03-03 | procedimentos e fármacos | RF-G03-05 e RF-G03-06 (divididos) |
| RF-G03-04 | pré-admissão com prioridade herdada | RF-G03-09 e RF-G03-10 (divididos) |
| RF-G03-05 | recusa com assinatura e testemunha | RF-G03-12 e RF-G03-13 (divididos) |
| RF-G03-06 | ficha offline e sincronização | RF-G03-07 e RF-G03-08 (divididos) |
| RF-G03-07 | acesso por perfil e registro de acesso | RF-G03-14 e RF-G03-15 (divididos) |
| RF-G03-08 | publicar dois eventos | RF-G03-02 e RF-G03-15 (cada evento junto do seu requisito) |

Requisitos novos, sem semente: RF-G03-03 (abertura da ficha por C2), RF-G03-10 (validação)
e RF-G03-11 (resposta de G4).

## 2. Requisitos Não Funcionais

Categorias aceitas: desempenho, segurança, disponibilidade, usabilidade, conformidade, manutenibilidade.
A coluna **Origem do alvo** diz de onde vem o número, para ninguém confundir premissa com fato.
Alvo marcado como *premissa do grupo* deve ser validado no M3 (testes e protótipos).

| ID | Categoria | Requisito | Métrica e alvo | Como será verificado | Origem do alvo |
| --- | --- | --- | --- | --- | --- |
| RNF-G03-01 | desempenho | Sincronização rápida após a reconexão. | Ficha sincronizada em ≤ 5 s no percentil 95, para fichas de até 30 registros de sinais vitais, 20 procedimentos e 10 fármacos. | Caso de teste de fronteira com a ficha no tamanho máximo. | 5 s: seed do professor. Tamanho da ficha: premissa do grupo. |
| RNF-G03-02 | disponibilidade | Nenhum dado clínico perdido por queda de conexão. | 0 registros perdidos em 100% dos cenários de queda: antes de salvar, durante o salvamento e durante a sincronização. | Casos de teste de falha (um por cenário) e análise da sequência de gravação local. | Seed do professor (reclassificada: o README a chamava de "confiabilidade", categoria que o E2 não lista). |
| RNF-G03-03 | segurança | Dado clínico protegido em repouso e em trânsito. | 100% dos campos classificados como dado clínico no dicionário de dados (E4) cifrados em repouso e em trânsito. 0 campos clínicos em texto claro numa inspeção do dicionário. | Inspeção campo a campo do dicionário de dados, no M2. | Lei 13.709/2018, art. 5º, II (dado de saúde é sensível). Res. CFM 2.314/2022, art. 3º e art. 10, §2º. |
| RNF-G03-04 | usabilidade | Ficha utilizável em tablet com uma mão, em campo. | 100% dos controles interativos com alvo de toque ≥ 44 × 44 px. Registro de um conjunto de sinais vitais em ≤ 45 s e ≤ 8 toques, com 3 testadores. | Medição no wireframe (M3) e teste de usabilidade com 3 pessoas. | 44 px: seed do professor (coincide com WCAG 2.5.5). 45 s e 8 toques: premissa do grupo. |
| RNF-G03-05 | conformidade | Registro de telemedicina com os campos de identificação exigidos pela norma. | 100% das teleorientações e documentos emitidos trazem nome e CRM do médico, data e hora e a indicação de que foram emitidos por telemedicina. | Inspeção do dicionário de dados (M2) e caso de teste de conformidade (M3). | Res. CFM 2.314/2022, art. 13 (identificação do médico, data/hora, indicação de telemedicina). O grupo deve reler o artigo antes do M3. |
| RNF-G03-06 | desempenho | Pedido de pré-admissão sai rápido, porque o prazo de reserva depende dele. | C3 publicado em ≤ 3 s no percentil 95 após a confirmação do regulador, com conexão disponível, a 20 pedidos por minuto. | Análise da sequência (M2) e caso de teste de fronteira (M3). | Premissa do grupo (valor de referência: exemplo de 20 req/min do professor). |
| RNF-G03-07 | manutenibilidade | Contratos de integração estáveis e rastreáveis. | 100% dos campos publicados em C3, C4 e C5 correspondem ao contrato vigente. Mudança de campo só entra com ADR. | Seção "Contratos" da matriz de rastreabilidade, conferida a cada PR. | ARQ (§1, §6). |
| RNF-G03-08 | segurança | Sigilo da ficha em tablet exposto em campo. | Bloqueio automático após ≤ 2 min de inatividade. Retomada exige reautenticação. Nenhum dado de paciente aparece na tela bloqueada. | Caso de teste de fronteira (M3) e revisão do wireframe da tela de bloqueio. | RN-11. O valor de 2 min é premissa do grupo. |

## 3. Perguntas de domínio do G3: respostas iniciais

O professor listou três perguntas que o grupo precisa responder. As respostas abaixo são
de M1 e serão refinadas no M2 (com ADR quando a decisão for estrutural).

| Pergunta | Resposta inicial | Requisitos |
| --- | --- | --- |
| O que é obrigatório registrar antes da chegada? | Pelo menos um registro de sinais vitais com horário de coleta, a prioridade herdada, a necessidade de leito e o ETA ao hospital. Sem eles, o C3 não sai. | RF-G03-04, RF-G03-09, RF-G03-10 |
| O que fazer se a rede cair no local? | A ficha continua utilizável. Cada registro fica local com estado "pendente" e sincroniza depois sem duplicar. O pedido de pré-admissão fica "pendente de envio" e alerta a equipe. | RF-G03-07, RF-G03-08, RNF-G03-01, RNF-G03-02 |
| Como garantir sigilo na ficha? | Acesso por perfil e finalidade, evento para cada acesso (inclusive negado), dado clínico cifrado e bloqueio por inatividade. | RF-G03-14, RF-G03-15, RNF-G03-03, RNF-G03-08 |
