# UC-G03-04 — Registrar Recusa de Atendimento

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | UC-G03-04 |
| **Módulo** | G03 — Telemedicina e Prontuário Pré-Hospitalar |
| **Ator primário** | EquipeIntervencao |
| **Atores secundários** | MedicoRegulador (ciência da recusa), Módulo de Auditoria (G6, dono da assinatura digital) |
| **Participante fora do sistema** | Paciente (ou representante legal) e testemunha: assinam, mas não operam o sistema |
| **Objetivo** | Registrar com evidência a recusa de atendimento ou transporte por paciente consciente, protegendo o paciente e o serviço |
| **Requisitos rastreados** | RF-G03-12, RF-G03-13 |
| **Prioridade** | essencial |

## Pré-condições

- Ficha aberta para o atendimento (UC-G03-02 em andamento).
- A equipe avaliou o paciente como consciente e o informou dos riscos da recusa.

## Pós-condições

- **Sucesso:** `RecusaAtendimento` em "assinada", vinculada ao `prontuario_id`, com assinatura do paciente ou representante, testemunha identificada e ciência do médico regulador. Evento `RECUSA_ASSINADA` publicado para G6. A ficha fica sinalizada e a pré-admissão deixa de ser oferecida.
- **Falha:** recusa permanece em "rascunho" ou "não assinada"; nenhum evento `RECUSA_ASSINADA` é publicado.

## Fluxo principal

1. Durante o preenchimento, a equipe aciona "Registrar recusa de atendimento".
2. O sistema solicita a confirmação de que o paciente está consciente e a descrição dos riscos informados a ele.
3. A equipe confirma a consciência e registra os riscos informados e o motivo da recusa.
4. O sistema solicita a identificação da testemunha (nome e vínculo).
5. A equipe registra a testemunha.
6. O paciente, ou o representante legal, assina a recusa.
7. A testemunha assina.
8. O sistema solicita a ciência do médico regulador; o regulador assina a ciência.
9. O sistema verifica que as três evidências estão presentes (RN-10).
10. O sistema grava a recusa como "assinada", vincula ao prontuário e sinaliza a ficha.
11. O sistema publica o evento `RECUSA_ASSINADA` para G6, com o motivo como justificativa.

## Fluxos alternativos

### FA-01 — Representante legal assina pelo paciente

- **Condição:** o paciente é menor ou incapaz e há um representante legal presente.
- **Passos:** 1. A equipe registra o vínculo do representante. 2. O representante assina no passo 6.
- **Reconvergência:** retorna ao passo 7 do fluxo principal.

### FA-02 — Paciente recusa-se a assinar

- **Condição:** o paciente mantém a recusa de atendimento, mas não quer assinar.
- **Passos:** 1. A equipe registra que o paciente não assinou, com o motivo. 2. A testemunha assina e o regulador toma ciência. 3. O sistema grava a recusa como **"não assinada"**, estado distinto de "assinada".
- **Estado final:** a RN-10 exige assinatura; esse estado não a satisfaz e não publica `RECUSA_ASSINADA`. O tratamento é proposta do grupo, a decidir em ADR (ver Q2).

### FA-03 — Paciente muda de ideia

- **Condição:** depois do registro, o paciente aceita o atendimento.
- **Passos:** 1. A equipe registra a revogação como novo registro, referenciando a recusa. 2. O sistema reabre a oferta de pré-admissão.
- **Estado final:** recusa original preservada no histórico.

## Fluxos de exceção

### FE-01 — Evidência incompleta

- **Condição:** falta assinatura, testemunha ou ciência no passo 9.
- **Passos:** 1. O sistema não conclui a recusa. 2. O sistema indica a evidência faltante.
- **Estado final:** recusa em "rascunho"; nenhum evento publicado.

### FE-02 — Consciência não confirmada

- **Condição:** a equipe não confirma a consciência do paciente no passo 3.
- **Passos:** 1. O sistema impede o registro de recusa. 2. O sistema orienta a continuidade do atendimento.
- **Estado final:** nenhuma recusa criada. A RN-10 vale apenas para paciente consciente.

### FE-03 — Sem conexão ao coletar as assinaturas

- **Condição:** o dispositivo está offline.
- **Passos:** 1. O sistema guarda localmente as assinaturas e a recusa, com o horário do fato. 2. Ao reconectar, UC-G03-08 sincroniza e o evento é publicado.
- **Estado final:** recusa "assinada" com horário original; evento publicado só após a sincronização.

## Regras de negócio aplicadas

| Regra | Descrição |
| --- | --- |
| RN-10 | passos 6 a 9: recusa de paciente consciente exige assinatura e testemunha |
| RN-09 | passo 11: ação sensível gera evento imutável |
| RN-11 | a recusa contém dado clínico e fica sob o mesmo controle de acesso da ficha |

## Relacionamentos

- `<<extend>>` **UC-G03-02 Preencher Ficha Pré-Hospitalar**: extensão opcional, disparada só quando um paciente consciente recusa o atendimento. Ponto de extensão: "recusa de atendimento".

## Dados manipulados

| Entidade | Dado | Origem |
| --- | --- | --- |
| RecusaAtendimento | motivo, riscos informados, estado, testemunha, ciência | este módulo (entidade própria, ADR-G03-04) |
| AssinaturaDigital | evidência de autoria | G6 (por referência) |
| Prontuario | `prontuario_id`, sinal de recusa | este módulo |

## Critérios de aceite

- [ ] Recusa "assinada" só existe com assinatura do paciente ou representante, testemunha e ciência do regulador.
- [ ] Sem as três evidências, nenhum evento `RECUSA_ASSINADA` é publicado.
- [ ] Recusa de paciente sem consciência confirmada não pode ser registrada.
- [ ] A revogação nunca apaga a recusa original.

## Rastreabilidade

| RF | Classe | Sequência | Teste |
| --- | --- | --- | --- |
| RF-G03-12 | RecusaAtendimento | DS-G03-05, ME-G03-03, DA-G03-02 | CT-G03-05 |
| RF-G03-13 | RecusaAtendimento, PublicacaoEvento | DS-G03-05 | CT-G03-05 |
