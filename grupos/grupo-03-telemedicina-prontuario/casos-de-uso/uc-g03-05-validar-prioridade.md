# UC-G03-05 — Validar Prioridade

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | UC-G03-05 |
| **Módulo** | G03 — Telemedicina e Prontuário Pré-Hospitalar |
| **Ator primário** | Sistema (acionado por `<<include>>` de UC-G03-03) |
| **Atores secundários** | G1 (dono da prioridade), G2 (origem do C2) |
| **Objetivo** | Garantir que o C3 carregue a prioridade vigente, herdada e somente leitura, com os campos obrigatórios completos |
| **Requisitos rastreados** | RF-G03-09, RF-G03-10 |
| **Prioridade** | essencial |

## Pré-condições

- UC-G03-03 em execução, com `chamado_id` e ficha disponíveis.
- C2 `despacho_ativo` recebido com `prioridade` e `prioridade_atualizada_em` (ADR-G03-01, proposto).

## Pós-condições

- **Sucesso:** prioridade vigente confirmada e copiada para a pré-admissão, sem edição; campos obrigatórios completos.
- **Falha:** envio bloqueado e campo faltante indicado; nenhum valor é digitado ou estimado neste módulo.

## Fluxo principal

1. O sistema lê `prioridade` e `prioridade_atualizada_em` do `Prontuario` (valores recebidos no C2).
2. O sistema verifica que o valor pertence ao conjunto de Manchester do contrato C1.
3. O sistema verifica `necessidade` em `SALA_VERMELHA` ou `OBSERVACAO`, presença de sinal vital e `eta_hospital`.
4. O sistema entrega a prioridade ao chamador, somente leitura, com o horário da classificação.

## Fluxos alternativos

### FA-01 — Reclassificação após o despacho

- **Condição:** G1 reclassificou o chamado e reemitiu o C2 (ADR-G03-01).
- **Passos:** 1. O sistema usa a versão mais recente do C2. 2. Atualiza `prioridade_atualizada_em` no `Prontuario`.
- **Reconvergência:** retorna ao passo 2 do fluxo principal.

### FA-02 — Plano B sem o campo no C2

- **Condição:** o ADR-G03-01 não foi aceito e o C2 segue sem `prioridade`.
- **Passos:** 1. O sistema consulta G1 por `chamado_id`. 2. Usa a prioridade vigente devolvida, somente leitura.
- **Reconvergência:** retorna ao passo 2 do fluxo principal.
- **Dependência aberta:** consulta síncrona entre módulos (ARQ §6); registrar aceite no ADR-G03-01.

## Fluxos de exceção

### FE-01 — Prioridade ausente ou inválida

- **Condição:** o chamado não tem prioridade ou o valor não pertence ao conjunto de Manchester.
- **Passos:** 1. O sistema bloqueia o envio. 2. Informa o motivo ao regulador.
- **Estado final:** nenhuma prioridade entregue; nenhum valor estimado.

### FE-02 — Campo obrigatório faltante

- **Condição:** falta sinal vital, `necessidade` ou `eta_hospital`.
- **Passos:** 1. O sistema bloqueia o envio. 2. Indica o campo faltante.
- **Estado final:** C3 não publicado; ficha permanece disponível para correção.

### FE-03 — G1 indisponível no plano B

- **Condição:** a consulta de fallback não responde.
- **Passos:** igual a FE-01, com motivo "G1 indisponível". O sistema não usa valor antigo de memória.
- **Estado final:** pedido não emitido; pendência visível ao regulador.

## Regras de negócio aplicadas

| Regra | Aplicação |
| --- | --- |
| RN-02 | a prioridade é de G1; este módulo só herda e copia no envio |
| Regra 4 de coerência | nenhum ator do G03 recalcula ou edita prioridade |
| ADR-G03-01 | prioridade no C2, com reemissão em cada reclassificação |

## Relacionamentos

- `<<include>>` por **UC-G03-03 Solicitar Pré-Admissão**: obrigatório, pois o C3 exige prioridade herdada e campos completos (RF-G03-10).

## Dados manipulados

| Entidade | Dado | Origem |
| --- | --- | --- |
| Prontuario | `prioridade_herdada`, `prioridade_atualizada_em` | C2 (G2), com G1 como dono |
| PreAdmissao | `prioridade` (cópia no envio), `necessidade`, `eta_hospital` | este módulo |

## Critérios de aceite

- [ ] O valor enviado no C3 é idêntico ao vigente recebido pelo C2.
- [ ] Nenhum caminho do módulo permite digitar ou recalcular prioridade.
- [ ] Ausência de sinal vital, `eta_hospital` ou prioridade bloqueia o pedido.

## Rastreabilidade

| RF | Classe | Sequência | Teste |
| --- | --- | --- | --- |
| RF-G03-09 | PreAdmissao, Prontuario | DS-G03-02, DA-G03-01 | CT-G03-01 |
| RF-G03-10 | PreAdmissao, Prontuario | DS-G03-02, DA-G03-01 | CT-G03-02 |
