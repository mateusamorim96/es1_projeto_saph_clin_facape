# G03 — Telemedicina e Prontuário Pré-Hospitalar

> Grupo responsável pelo subprocesso **Teleorientação, ficha pré-hospitalar digital e pré-admissão** do SAPH-Clin.
> Módulo: M2 — Telemedicina Regulada e Prontuário Integrado
> Entrega atual: **M1 v1.0** (E1, E2, E3) e modelagem do **M2** (E4, E5 e ADRs)

## Integrantes

| Nome | Matrícula | Papel no grupo (proposta, o grupo ajusta) |
| --- | --- | --- |
| Luccas Amaro de Castro Alencar | 26249 | requisitos e escopo (E1, E2) |
| Tulio Alves Torres | 26831 | casos de uso e diagramas (E3) |
| Geliton Fernandes de Almeida Filho | 26643 | rastreabilidade, testes e relatório (E6, E7) |

Cada integrante deve ter commits reais no PR (regra de avaliação do professor).

## E1 · Visão e escopo do módulo

**Problema que o módulo resolve.** A informação clínica produzida na rua não chega estruturada ao
hospital: a recepção redigita tudo na chegada e a reserva do leito crítico atrasa. O módulo M2 registra
o que o médico regulador orienta, o que a equipe mede e faz em campo (ficha pré-hospitalar digital) e
transforma esse registro em um pedido de pré-admissão enviado **antes** de a ambulância chegar.

**Onde o módulo começa.** Quando existe um chamado classificado com despacho ativo: o contrato C2
`despacho_ativo` (de G2) abre a ficha. A teleorientação sem viatura também começa aqui (ver Q3).

**Onde o módulo termina.**
- Ao publicar o C3 `pedido_pre_admissao` para G4 e registrar a resposta (reservado ou negado).
- Ao publicar o C4 `consumo_farmaco` para G5, uma vez por fármaco administrado.
- Ao publicar os eventos sensíveis (C5) para G6: `TELEORIENTACAO_REGISTRADA`, `RECUSA_ASSINADA` e `PRONTUARIO_ACESSADO`.
- Ao disponibilizar a ficha consolidada à `EquipeReceptora` (passagem de plantão digital).

**Atores.** `MedicoRegulador`, `EquipeIntervencao`, `EquipeReceptora`.

**Entidades do módulo.** `Teleorientacao`, `Prontuario`, `RegistroSinaisVitais`, `Procedimento`,
`ConsumoFarmaco`, `PreAdmissao`. Possível entidade adicional `RecusaAtendimento`, a confirmar no M2.

**Fora do escopo.**
- Decidir o despacho (G2) e a prioridade clínica (G1): este módulo herda a prioridade e nunca a recalcula.
- Aceitar ou negar a reserva de leito (G4) e dar baixa no estoque (G5).
- Criar trilha de auditoria própria ou a assinatura digital (G6): este módulo publica eventos e referencia a assinatura.
- Prontuário hospitalar definitivo, prescrição intra-hospitalar, faturamento e implementação real de APIs ou bancos.

**Premissas adotadas.**

| ID | Premissa | Fonte ou motivo |
| --- | --- | --- |
| P1 | A prioridade do chamado chega ao G3 pelo C2 e é somente leitura. Hoje o C2 não a carrega; a proposta de incluí-la está no ADR-G03-01. Se for recusada, o plano B é consultar G1 por `chamado_id`. | ARQ, C2 e C3; Q1; ADR-G03-01 |
| P2 | O `eta_hospital` é informado pela equipe ou pelo regulador, pois nenhum contrato o fornece. | ARQ, C2 só traz `eta_local`; ver Q2 |
| P3 | A recusa exige três evidências: assinatura do paciente (ou representante), testemunha e ciência do médico regulador. | RN-10 e DEF (Módulo 4); ver Q4 |
| P4 | G4 responde ao C3 com "reservado" ou "negado". O contrato só descreve explicitamente o "negado". | ARQ, C3 |
| P5 | O consentimento do paciente à telemedicina é dispensado em emergência médica. | Res. CFM 2.314/2022, art. 15, parágrafo único (conferido em fonte oficial; o grupo deve reler o artigo) |
| P6 | A Res. CFM 2.314/2022 não define "teleorientação". O grupo trata a teleorientação como ato de telemedicina registrado pelo médico regulador e aplica os campos do art. 13. | Res. CFM 2.314/2022; decisão do grupo, a registrar em ADR no M2 |

**Questões em aberto (a levar ao professor ou a decidir em ADR no M2).**

| ID | Questão | Impacto |
| --- | --- | --- |
| Q1 | O C2 não traz `prioridade`, mas o C3 a exige "herdada da triagem". G3 consulta G1 por `chamado_id` ou o contrato C2 passa a incluir o campo? | Mudança de contrato exige ADR e aviso a G1 e G2. **Proposta:** [ADR-G03-01](adr/adr-g03-01-prioridade-herdada-no-c2.md). |
| Q2 | Quem fornece o `eta_hospital`? G2 só entrega `eta_local`. | O prazo da reserva em G4 depende desse campo. |
| Q3 | Como a teleorientação entra em G3 quando não há viatura (sem despacho)? Não há contrato G1/G2 → G3 para esse caso. | O contexto (pergunta 3) e a definição (diagrama de atividades) preveem o desvio para telemedicina. **Proposta:** [ADR-G03-05](adr/adr-g03-05-teleorientacao-vinculada-ao-chamado-com-despacho-opcional.md). |
| Q4 | De quem é a assinatura na recusa? A RN-10 diz só "assinatura e testemunha"; a definição cita a assinatura do médico regulador. E se o paciente se recusa a assinar? | Define os estados de `RecusaAtendimento` (FA-02 do UC-G03-04). **Proposta:** [ADR-G03-04](adr/adr-g03-04-recusa-de-atendimento-como-entidade-com-tres-evidencias.md). |
| Q5 | O envio da pré-admissão é ação sensível (RN-09), mas não há evento correspondente no catálogo de G6. Criar `PRE_ADMISSAO_SOLICITADA`? | Mudança de catálogo exige ADR e aviso a G6. |

**Respostas iniciais às perguntas de domínio do G3.** Resumo (detalhe em [`requisitos.md`](requisitos.md), seção 3):
- *O que registrar antes da chegada?* Sinais vitais com horário de coleta, prioridade herdada, necessidade de leito e ETA.
- *E se a rede cair?* A ficha continua utilizável offline e sincroniza sem duplicar; o pedido fica "pendente de envio".
- *Como garantir sigilo?* Acesso por perfil, evento para cada acesso, dado clínico cifrado e bloqueio por inatividade.

## Documentos entregues

| Artefato | Onde |
| --- | --- |
| E1 Visão e escopo | este arquivo |
| E2 Catálogo de requisitos (15 RF, 8 RNF) | [`requisitos.md`](requisitos.md) |
| E3 Diagrama de casos de uso (fonte e exportação) | [`diagramas/diagrama-casos-de-uso-g03.puml`](diagramas/diagrama-casos-de-uso-g03.puml), `.svg`, `.png` |
| E3 Catálogo de casos de uso, relacionamentos e justificativas | [`casos-de-uso/uc-g03-00-catalogo-e-relacionamentos.md`](casos-de-uso/uc-g03-00-catalogo-e-relacionamentos.md) |
| E3 Especificação textual (UC-G03-01 a 08) | [`casos-de-uso/`](casos-de-uso/) |
| E4 Diagrama de classes de domínio (9 classes, herança de `RegistroClinico`) | [`diagramas/diagrama-classes-g03.puml`](diagramas/diagrama-classes-g03.puml), `.svg`, `.png` |
| E4 Dicionário de dados | [`dicionario-de-dados.md`](dicionario-de-dados.md) |
| E5 Diagramas de sequência (6, com exceções e indisponibilidades) | [`diagramas/ds-g03-*.puml`](diagramas/) |
| E5 Máquinas de estado (3: ficha, pré-admissão, recusa) | [`diagramas/me-g03-*.puml`](diagramas/) |
| E5 Diagramas de atividades (2) | [`diagramas/da-g03-*.puml`](diagramas/) |
| E5 Índice, tabelas de transição e coerência | [`modelagem-comportamental.md`](modelagem-comportamental.md) |
| E6 ADRs (5: 3 propostos, 2 aceitos) | [`adr/`](adr/) |
| E6 Casos de teste (7, um de fronteira de duplicidade e um de tamanho máximo; ainda não executados) | [`testes/`](testes/) |
| E6 Matriz de rastreabilidade (RF → UC → classe → sequência → ADR → teste) | [`matriz-rastreabilidade.md`](matriz-rastreabilidade.md) |

## Eventos que o módulo publica para G6

- `TELEORIENTACAO_REGISTRADA`
- `PRONTUARIO_ACESSADO`
- `RECUSA_ASSINADA`

## Riscos de modelagem conhecidos

- Transformar a ficha em prontuário hospitalar completo (fora de escopo).
- Recusa sem evidência de assinatura (RN-10).
- Pré-admissão recalculando prioridade em vez de herdar.
- Tratar o consentimento de telemedicina como obrigatório em emergência (a norma o dispensa; ver P5).
- Bloquear o registro clínico por regra de estoque (P-04).
- Sobrescrever registros em vez de versionar (ADR-G03-02).

## Estado da entrega

| Artefato | M1 (Aula 18) | M2 (Aula 24) | M3 (Aula 27) |
| --- | --- | --- | --- |
| E1 Visão e escopo | entregue v1.0 | — | — |
| E2 Catálogo RF/RNF | entregue v1.0 | — | — |
| E3 Casos de uso | entregue v1.0 | — | — |
| E4 Classes + dicionário | — | entregue v1.0 | — |
| E5 Sequência + estados | — | entregue v1.0 | — |
| E6 ADRs, matriz e testes | — | ADRs, matriz e 7 testes (não executados) entregues v1.0 | testes dos RF-G03-01 a 06 e execução |
| E7 Protótipos + relatório | — | — | pendente |

Atualize esta tabela a cada PR: é o primeiro lugar onde o professor olha.

## Como rodar a validação local

```bash
python3 scripts/check_entrega.py --grupo grupo-03-telemedicina-prontuario --marco M1
python3 scripts/check_entrega.py --grupo grupo-03-telemedicina-prontuario --marco M2
```

## Estrutura da pasta

```text
grupo-03-telemedicina-prontuario/
├── README.md                  ← este arquivo (escopo, premissas, estado)
├── requisitos.md              ← catálogo de RF e RNF (E2)
├── dicionario-de-dados.md     ← dicionário das classes (E4)
├── modelagem-comportamental.md ← índice e transições de estado (E5)
├── matriz-rastreabilidade.md  ← RF → UC → Classe → Sequência → ADR → CT
├── casos-de-uso/              ← UC-G03-NN-*.md
├── diagramas/                 ← fonte (.puml) + export (.png/.svg): UC, classes, DS, ME, DA
├── prototipos/                ← wireframes (fonte + imagem), a partir do M3
├── adr/                       ← ADR-G03-NN-*.md
└── testes/                    ← CT-G03-NN-*.md (7 escritos; execução no M3)
```

## Relatório técnico (E7)

Será escrito no M3, com decisões, dificuldades, limitações conhecidas e o que o grupo faria diferente.
