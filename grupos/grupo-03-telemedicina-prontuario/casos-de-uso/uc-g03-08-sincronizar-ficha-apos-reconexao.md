# UC-G03-08 — Sincronizar Ficha após Reconexão

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | UC-G03-08 |
| **Módulo** | G03 — Telemedicina e Prontuário Pré-Hospitalar |
| **Ator primário** | EquipeIntervencao (beneficiária; o disparo é automático) |
| **Atores secundários** | Sistema de Estoque (G05), Sistema de Auditoria (G6) |
| **Objetivo** | Enviar à central os registros feitos sem conexão, sem perda e sem duplicidade |
| **Requisitos rastreados** | RF-G03-07, RF-G03-08, RNF-G03-01, RNF-G03-02 |
| **Prioridade** | essencial |

## Pré-condições

- Existem registros locais pendentes feitos durante período sem conexão.
- A conexão foi restabelecida.

## Pós-condições

- **Sucesso:** registros gravados na central com horário do fato preservado, sem duplicidade; eventos e consumos pendentes entregues.
- **Falha:** registros pendentes permanecem no dispositivo, íntegros.

## Fluxo principal

1. O sistema detecta a reconexão.
2. O sistema envia os registros pendentes em ordem cronológica do fato.
3. A central valida e grava cada registro, reconhecendo-o por identificador (idempotência).
4. O sistema marca cada registro como sincronizado no dispositivo.
5. O sistema entrega os eventos pendentes a G06 e os consumos pendentes a G05.
6. O sistema informa a equipe do término da sincronização.

## Fluxos alternativos

### FA-01 — Conexão cai durante a sincronização

- **Condição:** perda de conexão no meio do envio.
- **Passos:** 1. O sistema retém os registros não reconhecidos. 2. Na próxima reconexão, reenvia apenas esses.
- **Reconvergência:** retorna ao passo 2.

### FA-02 — Registro corrigido offline

- **Condição:** o mesmo registro teve versões diferentes antes e depois da sincronização.
- **Passos:** 1. O sistema mantém as versões em ordem de horário. 2. Nenhuma versão sobrescreve a outra.
- **Reconvergência:** retorna ao passo 3.

## Fluxos de exceção

### FE-01 — Registro rejeitado pela central

- **Condição:** por exemplo, o despacho foi encerrado durante o período offline.
- **Passos:** 1. O registro é retido localmente com o motivo. 2. A equipe e o médico regulador são alertados.
- **Estado final:** nenhum dado clínico é descartado (RNF-G03-02).

### FE-02 — Falha persistente

- **Condição:** a sincronização falha em tentativas repetidas.
- **Passos:** 1. Os dados permanecem no dispositivo. 2. O sistema exibe alerta de pendência.
- **Estado final:** pendência visível; dados preservados.

## Regras de negócio aplicadas

| Regra | Aplicação |
| --- | --- |
| RN-09 | passo 5: eventos pendentes são entregues sem perda |
| RN-12 | passo 2: o horário do fato é preservado |

## Relacionamentos

- `<<extend>>` **UC-G03-02 Preencher Ficha Pré-Hospitalar** (ponto de extensão PE-2): opcional, porque só ocorre se houve período sem conexão.

## Dados manipulados

| Entidade | Dado | Origem |
| --- | --- | --- |
| Registros da ficha | sinais vitais, procedimentos, fármacos, recusa | este módulo, fila local |
| Eventos e consumos | `evento_id`, C4 | este módulo |

## Critérios de aceite

- [ ] Ficha sincronizada em até 5 s após a reconexão (RNF-G03-01).
- [ ] Reenvio do mesmo registro não cria duplicata.
- [ ] Horário do fato é preservado.
- [ ] Nenhum registro é descartado por rejeição ou falha.

## Rastreabilidade

| RF | Classe | Sequência | Teste |
| --- | --- | --- | --- |
| RF-G03-07 | RegistroClinico | DS-G03-04 | CT-G03-04 |
| RF-G03-08 | RegistroClinico | DS-G03-04, ME-G03-01 | CT-G03-04 |
