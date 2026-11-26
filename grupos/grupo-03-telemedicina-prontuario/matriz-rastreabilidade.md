# Matriz de Rastreabilidade — G03

> Modelo em `docs/templates/matriz-rastreabilidade.md`. Célula sem informação é pendência: marcada como `pendente`.
> Versão M2: caso de uso, classe, sequência e ADR completos; 7 casos de teste escritos (não executados), com RF-G03-01 a 06 ainda sem teste.
> Requisitos em [`requisitos.md`](requisitos.md); classes em [`dicionario-de-dados.md`](dicionario-de-dados.md);
> sequências e estados em [`modelagem-comportamental.md`](modelagem-comportamental.md).

## Requisitos funcionais

| RF | Descrição curta | Caso de uso | Classe(s) | Sequência / Estado | ADR | Caso de teste | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| RF-G03-01 | Registrar teleorientação | UC-G03-01 | Teleorientacao | DS-G03-01 | ADR-G03-05 | pendente | modelado (M2) |
| RF-G03-02 | Publicar `TELEORIENTACAO_REGISTRADA` | UC-G03-01 | Teleorientacao, PublicacaoEvento | DS-G03-01 | — | pendente | modelado (M2) |
| RF-G03-03 | Abrir ficha ao receber C2 | UC-G03-02 | Prontuario | DS-G03-04, ME-G03-01 | ADR-G03-01 | pendente | modelado (M2) |
| RF-G03-04 | Registrar sinais vitais com histórico | UC-G03-02 | RegistroClinico, RegistroSinaisVitais | DS-G03-04 | ADR-G03-02 | pendente | modelado (M2) |
| RF-G03-05 | Registrar procedimentos | UC-G03-02 | RegistroClinico, Procedimento | DS-G03-04 | ADR-G03-02 | pendente | modelado (M2) |
| RF-G03-06 | Registrar fármaco e publicar C4 | UC-G03-06 | RegistroClinico, ConsumoFarmaco | DS-G03-04 | ADR-G03-02 | pendente | modelado (M2) |
| RF-G03-07 | Ficha disponível offline | UC-G03-02, UC-G03-08 | RegistroClinico | DS-G03-04 | ADR-G03-03 | CT-G03-04 | modelado, com teste (M2) |
| RF-G03-08 | Sincronizar sem duplicar | UC-G03-08 | RegistroClinico | DS-G03-04, ME-G03-01 | ADR-G03-03 | CT-G03-04 | modelado, com teste (M2) |
| RF-G03-09 | Emitir pedido de pré-admissão (C3) | UC-G03-03 | PreAdmissao, Prontuario, RegistroSinaisVitais | DS-G03-02, ME-G03-02, DA-G03-01 | ADR-G03-01 | CT-G03-01 | modelado, com teste (M2) |
| RF-G03-10 | Validar prioridade e completude | UC-G03-05 | PreAdmissao, Prontuario | DS-G03-02, DA-G03-01 | ADR-G03-01 | CT-G03-01, CT-G03-02 | modelado, com teste (M2) |
| RF-G03-11 | Tratar resposta de G4 | UC-G03-03 | PreAdmissao | DS-G03-03, ME-G03-02 | — | CT-G03-03 | modelado, com teste (M2) |
| RF-G03-12 | Registrar recusa com três evidências | UC-G03-04 | RecusaAtendimento, Prontuario | DS-G03-05, ME-G03-03, DA-G03-02 | ADR-G03-04 | CT-G03-05 | modelado, com teste (M2) |
| RF-G03-13 | Publicar `RECUSA_ASSINADA` | UC-G03-04 | RecusaAtendimento, PublicacaoEvento | DS-G03-05 | ADR-G03-04 | CT-G03-05 | modelado, com teste (M2) |
| RF-G03-14 | Restringir acesso por perfil | UC-G03-07 | Prontuario | DS-G03-06 | — | CT-G03-06 | modelado, com teste (M2) |
| RF-G03-15 | Registrar acesso e publicar `PRONTUARIO_ACESSADO` | UC-G03-07 | Prontuario, PublicacaoEvento | DS-G03-06 | — | CT-G03-06 | modelado, com teste (M2) |

## Requisitos não funcionais

| RNF | Métrica e alvo | Onde aparece no modelo | Como é verificado | Status |
| --- | --- | --- | --- | --- |
| RNF-G03-01 | Sincronização ≤ 5 s (p95), ficha de até 30 + 20 + 10 registros | DS-G03-04; ADR-G03-03 | CT-G03-07 (escrito, não executado) | pendente |
| RNF-G03-02 | 0 registros perdidos em 3 cenários de queda | DS-G03-04; `RegistroClinico.sincronizacao`; ADR-G03-03 | CT-G03-04 (escrito, não executado) | pendente |
| RNF-G03-03 | 100% dos campos clínicos cifrados | dicionário de dados (campos clínicos identificados) | inspeção do dicionário e teste (M3) | pendente |
| RNF-G03-04 | alvo de toque ≥ 44 px; sinais vitais em ≤ 45 s e ≤ 8 toques | wireframes (M3) | wireframe e teste de usabilidade (M3) | pendente |
| RNF-G03-05 | 100% dos registros com nome e CRM, data/hora e indicação de telemedicina | `Teleorientacao.medico_nome`, `medico_crm`, `emitida_por_telemedicina` | inspeção do dicionário e teste (M3) | atendido no modelo |
| RNF-G03-06 | C3 publicado em ≤ 3 s (p95) a 20 pedidos/min | DS-G03-02, DA-G03-01 | caso de teste (M3) | pendente |
| RNF-G03-07 | 100% dos campos de C3, C4 e C5 conforme o contrato | seção "Mapa de contratos" do dicionário | conferência a cada PR | atendido no modelo |
| RNF-G03-08 | bloqueio por inatividade em ≤ 2 min | FE-04 do UC-G03-02; ADR-G03-03 | caso de teste e wireframe (M3) | pendente |

## Casos de uso → diagramas

| UC | Sequência | Estado | Atividade |
| --- | --- | --- | --- |
| UC-G03-01 | DS-G03-01 | — | — |
| UC-G03-02 | DS-G03-04 | ME-G03-01 | — |
| UC-G03-03 | DS-G03-02, DS-G03-03 | ME-G03-01, ME-G03-02 | DA-G03-01 |
| UC-G03-04 | DS-G03-05 | ME-G03-03 | DA-G03-02 |
| UC-G03-05 | DS-G03-02 | — | DA-G03-01 |
| UC-G03-06 | DS-G03-04 | — | — |
| UC-G03-07 | DS-G03-06 | — | — |
| UC-G03-08 | DS-G03-04 | — | — |

## Decisões (ADR)

| ADR | Decisão | Status | Questão que resolve |
| --- | --- | --- | --- |
| [ADR-G03-01](adr/adr-g03-01-prioridade-herdada-no-c2.md) | A prioridade chega no C2 e é somente leitura | proposto | Q1 |
| [ADR-G03-02](adr/adr-g03-02-registros-clinicos-imutaveis-e-versionados.md) | Registros clínicos imutáveis e versionados | aceito | decisão interna |
| [ADR-G03-03](adr/adr-g03-03-ficha-offline-first-com-sincronizacao-idempotente.md) | Ficha offline-first com sincronização idempotente | aceito | decisão interna |
| [ADR-G03-04](adr/adr-g03-04-recusa-de-atendimento-como-entidade-com-tres-evidencias.md) | Recusa como entidade com três evidências | proposto | Q4 |
| [ADR-G03-05](adr/adr-g03-05-teleorientacao-vinculada-ao-chamado-com-despacho-opcional.md) | Teleorientação vinculada ao chamado, despacho opcional | proposto | Q3 |

## Contratos consumidos

| Contrato | Origem | Uso no módulo | Campo crítico |
| --- | --- | --- | --- |
| C2 `despacho_ativo` | G2 | abre a ficha (RF-G03-03) | `despacho_id`, `chamado_id`, `equipe_id`; `prioridade` se o ADR-G03-01 for aceito |
| `sem_viatura_disponivel` (proposto) | G2 | teleorientação sem despacho (ADR-G03-05) | `chamado_id`, motivo |
| Resposta ao C3 (`reservado` ou `negado`) | G4 | atualiza a pré-admissão (RF-G03-11) | motivo do `negado`, `reserva_ref` |
| `registrarAssinatura` (proposto) | G6 | evidências da recusa (ADR-G03-04) | `assinatura_id` |

## Contratos fornecidos

| Contrato | Destino | Campo crítico | Estabilidade |
| --- | --- | --- | --- |
| C3 `pedido_pre_admissao` | G4 | `prontuario_id`, `prioridade`, `necessidade`, `eta_hospital` | estável, com Q2 em aberto |
| C4 `consumo_farmaco` | G5 | `lote`, `item_id`, `prontuario_id` | estável |
| C5 `evento_sensivel` (3 tipos) | G6 | `evento_id`, `tipo`, `ator_id`, `resultado` | estável, com Q5 em aberto |

## Rastreio a indicadores de G06 (bônus)

| RF | Indicador de G06 | Como o requisito alimenta o indicador |
| --- | --- | --- |
| RF-G03-11 | Taxa de reserva negada | cada pedido e seu desfecho ficam registrados e contáveis |
| RF-G03-09 | Tempo porta-leito | horário de envio do pedido e ETA ao hospital |
| RF-G03-01 | Tempo de resposta | horário do fato da teleorientação |
| RF-G03-15 | Auditoria de acesso a prontuário | cada acesso gera evento `PRONTUARIO_ACESSADO` |

## Pendências e riscos

| Item | Impacto | Prazo interno |
| --- | --- | --- |
| Aceite de G1 e G2 (ADR-G03-01), G2 (ADR-G03-05) e G6 (ADR-G03-04) | mudam contratos; sem aceite, vale o plano B de cada ADR | antes do M3 |
| Q2 (origem do `eta_hospital`) e Q5 (evento da pré-admissão) | sem ADR ainda | M3 |
| Casos de teste dos RF-G03-01 a 06 e dos RNF restantes; protótipos | colunas `pendente` até o M3 (o E6 pede no mínimo 4 testes, já há 7) | M3 |
| Releitura dos artigos citados da Res. CFM 2.314/2022 | RNF-G03-05 e premissas P5 e P6 | antes do M3 |

## Checklist final

- [x] Cada RF tem caso de uso.
- [x] Cada caso de uso tem classe.
- [x] Cada fluxo crítico tem diagrama de sequência.
- [ ] Cada RF tem pelo menos um caso de teste (cobertos: RF-G03-07 a 15; faltam RF-G03-01 a 06, para o M3).
- [ ] Nenhum `pendente` no M3.
- [x] Contratos consumidos e fornecidos conferem com [`arquitetura-modular.md`](../../docs/arquitetura-modular.md), com as lacunas Q1, Q2, Q3 e Q5 declaradas e as propostas marcadas como tais.
