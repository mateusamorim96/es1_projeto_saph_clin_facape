# UC-G03-02 — Preencher Ficha Pré-Hospitalar

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | UC-G03-02 |
| **Módulo** | G03 — Telemedicina e Prontuário Pré-Hospitalar |
| **Ator primário** | EquipeIntervencao |
| **Atores secundários** | MedicoRegulador (leitura em tempo real), Módulo de Auditoria (G6) |
| **Objetivo** | Registrar em campo, de forma estruturada, os sinais vitais, procedimentos e medicações do atendimento, mesmo sem conexão |
| **Requisitos rastreados** | RF-G03-03, RF-G03-04, RF-G03-05, RF-G03-07, RNF-G03-02, RNF-G03-04, RNF-G03-08 |
| **Prioridade** | essencial |

## Pré-condições

- O contrato C2 `despacho_ativo` foi recebido e a ficha foi aberta (RF-G03-03).
- O usuário pertence à equipe do despacho (`equipe_id`) e está autenticado.
- O dispositivo (tablet) tem a ficha disponível localmente.

## Pós-condições

- **Sucesso:** ficha com os registros do atendimento, cada um com autor e horário. Registros sincronizados, ou guardados localmente como "pendentes de sincronização".
- **Falha:** nenhum registro é descartado. Se a gravação local falhar, o sistema informa e não mostra o registro como salvo.

## Fluxo principal

1. A equipe de intervenção abre a ficha do atendimento em andamento.
2. O sistema exibe a ficha vinculada a `chamado_id` e `despacho_id`, com as teleorientações já recebidas.
3. A equipe registra os sinais vitais, informando os valores e o horário de coleta.
4. O sistema verifica tipo e unidade dos valores. Se um valor parecer implausível, pede confirmação, sem impedir o registro (a decisão clínica é da equipe).
5. A equipe registra os procedimentos realizados, com tipo, horário e responsável.
6. O sistema grava cada registro com autor e horário e preserva o histórico dos anteriores. <!-- ponto de extensão: fármaco administrado (UC-G03-06) -->
7. A equipe repete os passos 3 a 6 enquanto durar o atendimento.
8. O sistema mantém a ficha atualizada e disponível para consulta conforme o perfil (UC-G03-07).

## Fluxos alternativos

### FA-01 — Sem conexão durante o preenchimento (`<<extend>>` UC-G03-08)

- **Condição:** a conexão cai ou não existe no local.
- **Passos:** 1. O sistema sinaliza o modo offline sem interromper o preenchimento. 2. Cada registro é gravado localmente como "pendente de sincronização". 3. Ao reconectar, o sistema aciona UC-G03-08.
- **Reconvergência:** retorna ao passo em que a equipe estava.

### FA-02 — Fármaco administrado (`<<extend>>` UC-G03-06)

- **Condição:** a equipe administra um fármaco durante o atendimento (passo 6).
- **Passos:** 1. A equipe aciona o registro de fármaco. 2. O sistema executa UC-G03-06.
- **Reconvergência:** retorna ao passo 6.

### FA-03 — Paciente consciente recusa o atendimento (`<<extend>>` UC-G03-04)

- **Condição:** o paciente ou o representante legal recusa atendimento ou transporte.
- **Passos:** 1. A equipe aciona o registro de recusa. 2. O sistema executa UC-G03-04.
- **Reconvergência:** se a recusa for concluída, o fluxo termina. Se o paciente mudar de ideia, retorna ao passo 3.

### FA-04 — Correção de registro

- **Condição:** a equipe percebe um erro em registro já gravado.
- **Passos:** 1. A equipe seleciona o registro e informa o valor correto e o motivo. 2. O sistema cria nova versão e mantém a original visível no histórico, com o autor da correção.
- **Reconvergência:** retorna ao passo 6.

## Fluxos de exceção

### FE-01 — Falha de gravação local

- **Condição:** o dispositivo não consegue gravar o registro (por exemplo, sem espaço).
- **Passos:** 1. O sistema alerta de imediato. 2. O sistema não exibe o registro como salvo. 3. A equipe mantém o dado na tela até a gravação ser possível.
- **Estado final:** nenhum registro marcado como salvo sem estar de fato gravado (RNF-G03-02).

### FE-02 — Despacho encerrado ou cancelado durante o preenchimento

- **Condição:** o despacho muda de estado enquanto a equipe preenche, possivelmente offline.
- **Passos:** 1. Ao sincronizar, o sistema aceita os registros feitos antes do encerramento, pelo horário do fato. 2. O sistema sinaliza os registros posteriores para revisão do regulador.
- **Estado final:** nenhum registro clínico é descartado; os registros posteriores ficam marcados.

### FE-03 — Usuário fora da equipe do despacho

- **Condição:** o usuário tenta abrir a ficha sem pertencer ao `equipe_id`.
- **Passos:** 1. O sistema nega o acesso. 2. O sistema registra o acesso negado (RF-G03-15).
- **Estado final:** ficha inalterada, evento `PRONTUARIO_ACESSADO` com resultado `NEGADO`.

### FE-04 — Bloqueio por inatividade

- **Condição:** nenhuma interação por mais de 2 min (RNF-G03-08).
- **Passos:** 1. O sistema bloqueia a tela sem exibir dado do paciente. 2. A equipe se reautentica e retoma de onde parou.
- **Estado final:** nada perdido; o rascunho em edição é preservado.

## Regras de negócio aplicadas

| Regra | Descrição |
| --- | --- |
| RN-09 | passos 6 e FE-03: ação sensível e acesso geram evento |
| RN-11 | pré-condição e FE-03: só quem atende acessa a ficha |
| RN-07 | FA-02: fármaco só entra com lote (ver UC-G03-06) |

## Relacionamentos

- `<<extend>>` por **UC-G03-04 Registrar Recusa de Atendimento**: a maioria dos atendimentos não termina em recusa; só ocorre quando o paciente consciente recusa.
- `<<extend>>` por **UC-G03-06 Registrar Consumo de Fármaco**: a maioria dos atendimentos não administra fármaco; quando ocorre, o registro com lote passa a ser obrigatório.
- `<<extend>>` por **UC-G03-08 Sincronizar Ficha após Reconexão**: só ocorre quando há registros pendentes por falta de conexão.

## Dados manipulados

| Entidade | Dado | Origem |
| --- | --- | --- |
| Prontuario | `prontuario_id`, estado da ficha | este módulo |
| RegistroSinaisVitais | valores, horário de coleta, responsável | este módulo |
| Procedimento | tipo, horário, responsável | este módulo |
| Despacho | `despacho_id`, `equipe_id` | contrato C2 (G2) |

## Critérios de aceite

- [ ] Cada sinal vital tem horário de coleta, distinto do horário de digitação.
- [ ] Correção nunca apaga o valor original.
- [ ] Com a conexão desligada, a equipe consegue abrir a ficha e registrar.
- [ ] Nenhuma queda de conexão causa perda de registro.

## Rastreabilidade

| RF | Classe | Sequência | Teste |
| --- | --- | --- | --- |
| RF-G03-03 | pendente (M2) | pendente (M2) | pendente (M3) |
| RF-G03-04 | pendente (M2) | pendente (M2) | pendente (M3) |
| RF-G03-05 | pendente (M2) | pendente (M2) | pendente (M3) |
| RF-G03-07 | pendente (M2) | pendente (M2) | pendente (M3) |
