# G01 — Regulação e Triagem

> Grupo responsável pelo subprocesso **Acolhimento, classificação de risco e ciclo de vida do Chamado** do SAPH-Clin.
> Módulo: M1 — Regulação e Despacho de Urgência

## Integrantes

| Nome | Matrícula | Papel no grupo |
| --- | --- | --- |
| Gabriel Bezerra |26729| requisitos |
| Paulo Henrique |26633 | modelagem estrutural |
| Pedro Hebert |26581| modelagem comportamental |
| Isaque Bispo |26593| testes e rastreabilidade |

## Escopo do módulo (E1)

- **Subprocesso:** Acolhimento, classificação de risco e ciclo de vida do Chamado.
- **Atores envolvidos:** TASI, MedicoRegulador.
- **Entidades de domínio (donas do módulo):** `Chamado`, `Triagem`, `Solicitante`, `ClassificacaoRisco`.
- **Entrada:** Ligação do solicitante (fora do sistema).
- **Saída:** Contrato C1 `chamado_classificado` para G2.

### Fora do escopo do grupo

- escolher viatura ou equipe (G2).
- registrar dados clínicos de campo (G3).
- reservar leito (G4).

> Se algo do escopo acima for necessário, consuma o **contrato** descrito em
> `docs/arquitetura-modular.md` — não modele a entidade do vizinho.

## Casos de uso esperados (E3)

- UC-G01-01 Registrar Chamado
- UC-G01-02 Classificar Risco (Manchester)
- UC-G01-03 Reclassificar Chamado (`<<extend>>` de Classificar Risco)
- UC-G01-04 Cancelar Chamado

Use `docs/templates/caso-de-uso.md`. Mínimo: 4 casos de uso, sendo 2
especificados textualmente e pelo menos um `<<include>>` e um `<<extend>>`
justificados.

## Requisitos âncora (ponto de partida, não de chegada)

| ID | Requisito funcional | Caso de uso |
| --- | --- | --- |
| RF-G01-01 | Registrar chamado com solicitante, localização e queixa principal | UC-G01-01 |
| RF-G01-02 | Classificar risco nas cinco cores do Protocolo de Manchester | UC-G01-02 |
| RF-G01-03 | Registrar autor e justificativa em toda reclassificação | UC-G01-03 |
| RF-G01-04 | Detectar chamado duplicado da mesma ocorrência | UC-G01-01 |
| RF-G01-05 | Cancelar chamado com motivo obrigatório | UC-G01-04 |
| RF-G01-06 | Manter histórico completo de classificações do chamado | UC-G01-02 |
| RF-G01-07 | Alertar regulador quando triagem exceder 4 minutos (RN-01) | UC-G01-02 |
| RF-G01-08 | Publicar evento `CHAMADO_CLASSIFICADO` para G6 | UC-G01-02 |

| ID | Requisito não funcional (com métrica) | Categoria |
| --- | --- | --- |
| RNF-G01-01 | Acolhimento concluído em ≤ 90 s no fluxo principal | desempenho |
| RNF-G01-02 | Triagem disponível mesmo com falha de rede local (registro offline) | disponibilidade |
| RNF-G01-03 | Dado do solicitante protegido por perfil de acesso (LGPD) | segurança |
| RNF-G01-04 | Reclassificação visível no histórico em ≤ 3 s | desempenho |
| RNF-G01-05 | Vocabulário de queixa alinhado ao glossário do domínio | usabilidade |

> Estes requisitos são **sementes** extraídas do contexto. O grupo deve
> refiná-los, dividir os compostos e acrescentar o que a análise revelar.
> Requisito copiado sem análise perde nota em profundidade.

## Eventos que o módulo publica para G06

- `CHAMADO_CLASSIFICADO`
- `PRIORIDADE_RECLASSIFICADA`
- `CHAMADO_CANCELADO`

## Riscos de modelagem conhecidos

- modelar triagem como atributo de cor (perde histórico de reclassificação).
- aceitar classificação sem registro do médico regulador.
- esquecer que o TASI não decide conduta clínica.

## Estado da entrega

| Artefato | M1 (Aula 18) | M2 (Aula 24) | M3 (Aula 27) |
| --- | --- | --- | --- |
| E1 Visão e escopo | pendente | — | — |
| E2 Catálogo RF/RNF | pendente | — | — |
| E3 Casos de uso | pendente | — | — |
| E4 Classes + dicionário | — | pendente | — |
| E5 Sequência + estados | — | pendente | — |
| E6 ADRs, matriz e testes | — | parcial | pendente |
| E7 Protótipos + relatório | — | — | pendente |

Atualize esta tabela a cada PR: é o primeiro lugar onde o professor olha.

## Como rodar a validação local

```bash
python3 scripts/check_entrega.py --grupo grupo-01-regulacao-triagem
```

## Estrutura da pasta

```text
grupo-01-regulacao-triagem/
├── README.md                  ← este arquivo (escopo, requisitos, estado)
├── matriz-rastreabilidade.md  ← RF → UC → Classe → Sequência → CT
├── casos-de-uso/              ← UC-G0X-NN-*.md
├── diagramas/                 ← fonte (.puml/.drawio/.bpmn) + export (.png/.svg)
├── prototipos/                ← wireframes (fonte + imagem)
├── adr/                       ← ADR-G0X-NN-*.md
└── testes/                    ← CT-G0X-NN-*.md
```
