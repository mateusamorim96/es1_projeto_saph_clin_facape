Título do PR: `[G03] Entrega M2 — Telemedicina e Prontuário`

## 📦 PR do Grupo: G03 — Telemedicina e Prontuário

**Marco:** M2  ·  **Integrantes:**
- Luccas (matrícula: ___)
- Tulio (matrícula: ___)
- Geliton (matrícula: ___)

---

## 📋 Artefatos Entregues

| Arquivo | Artefato (E1–E7) | Descrição |
|---------|------------------|-----------|
| `diagramas/diagrama-classes-g03.puml` (+ `.svg`, `.png`) | E4 | 9 classes, herança de `RegistroClinico`, multiplicidades e enums |
| `dicionario-de-dados.md` | E4 | Significado, atributos, restrições e relacionamentos de cada classe; mapa de contratos |
| `diagramas/ds-g03-01` a `ds-g03-06` (`.puml`, `.svg`, `.png`) | E5 | 6 sequências; 4 tratam exceção ou indisponibilidade |
| `diagramas/me-g03-01` a `me-g03-03` | E5 | Máquinas de estado: ficha (ciclo central), pré-admissão e recusa |
| `diagramas/da-g03-01` e `da-g03-02` | E5 | Atividades: decisões da pré-admissão e da recusa |
| `modelagem-comportamental.md` | E5 | Índice e tabelas de transição |
| `adr/adr-g03-01` a `adr-g03-05` | E6 | 5 ADRs (2 aceitos, 3 propostos) |
| `testes/ct-g03-01` a `ct-g03-07` | E6 | 7 casos de teste (não executados), incluindo fronteira |
| `matriz-rastreabilidade.md` | E6 | RF → UC → classe → sequência/estado → ADR → teste |
| E1, E2, E3 (M1) | — | já entregues; E3 reexportado pelo PlantUML |

---

## 🎯 Decisões Tomadas

Modelamos `RegistroClinico` como classe abstrata com três subclasses (sinais vitais, procedimento e fármaco), porque
elas compartilham autoria, horário do fato, versionamento e estado de sincronização (ADR-G03-02, ADR-G03-03). O
horário do fato (`ocorrido_em`) é separado do horário do registro, pois a ficha é preenchida offline e sincronizada
depois. A prioridade é somente leitura no G3 e a `PreAdmissao` guarda só uma cópia no envio (ADR-G03-01). A recusa
virou entidade própria com estados e três evidências, e as assinaturas são referenciadas por identificador de G6, sem
duplicar a entidade (ADR-G03-04). A `PublicacaoEvento` é só uma fila de entrega de eventos a G6, nunca uma trilha de
auditoria paralela. O ciclo central é o da ficha (`Prontuario`), com guardas explícitas, como "encerrar só sem
registros pendentes". Três ADRs mudam contratos de outros grupos e por isso estão `proposto`, cada um com plano B:
C2 com `prioridade` (G1/G2), contrato `sem_viatura_disponivel` (G2) e `registrarAssinatura` (G6).

---

## ⚠️ Limitações e Itens em Aberto

- ADR-G03-01, 04 e 05 dependem de aceite de G1, G2 e G6; Q2 (`eta_hospital`) e Q5 (evento da pré-admissão) seguem sem ADR.
- Os 7 casos de teste estão escritos mas não executados; faltam testes dos RF-G03-01 a 06.
- Estado `NAO_ASSINADA` da recusa e os alvos numéricos (60 s, 2 min, 45 s, tamanho da ficha) são premissas do grupo.
- Falta a revisão do grupo sobre a Res. CFM 2.314/2022 (RNF-G03-05, P5 e P6).
- Protótipos (E7) e relatório ficam para o M3.

---

## 🔗 Rastreabilidade

- [x] Baseado em `docs/definicao-projeto.md`, `docs/contexto-do-sistema.md` e `docs/arquitetura-modular.md`
- [x] Consome contrato de outro grupo: C2 (G2); prioridade de G1; resposta de G4 ao C3
- [x] Fornece contrato para outro grupo: C3 (G4), C4 (G5), C5 (G6)
- [x] Propõe contratos novos: C2 com `prioridade`, `sem_viatura_disponivel`, `registrarAssinatura`

---

## 🤖 Uso de IA (obrigatório)

| Artefato | Ferramenta | O que o grupo revisou/validou |
|----------|------------|-------------------------------|
| Diagramas (.puml), dicionário, modelagem comportamental, ADRs, casos de teste, matriz | Claude (Anthropic) | **A preencher pelo grupo após a revisão:** conferir classes e transições contra os casos de uso, validar as premissas e discutir os ADRs propostos. |

---

## ✅ Checklist Obrigatório

- [X] Todos os arquivos estão dentro de `grupos/grupo-XX-*/`
- [X] Nenhum arquivo modificado fora da pasta do grupo
- [X] `python3 scripts/check_entrega.py --grupo grupo-03 --marco M2` passou localmente (passou na geração: APROVADO)
- [x] Diagramas têm fonte editável **e** exportação (`.png`/`.svg`)
- [x] Nomes de arquivo em kebab-case, sem acento e sem espaço
- [x] `matriz-rastreabilidade.md` atualizada
- [x] README do grupo com a tabela de estado da entrega atualizada
- [X] Commits no padrão `[G03] tipo: descrição`
- [ ] Cada integrante tem commits no PR
