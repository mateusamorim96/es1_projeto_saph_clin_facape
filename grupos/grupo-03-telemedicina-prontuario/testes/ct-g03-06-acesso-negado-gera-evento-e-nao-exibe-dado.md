# CT-G03-06 — Acesso fora do perfil é negado, registrado e não exibe dado clínico

## Identificação

| Campo | Valor |
| --- | --- |
| **ID** | CT-G03-06 |
| **Requisito rastreado** | RF-G03-14, RF-G03-15 |
| **Caso de uso** | UC-G03-07 |
| **Tipo** | exceção (segurança) |
| **Prioridade** | alta |

## Objetivo

Garante que o controle de perfil da RN-11 bloqueia quem não atende o paciente e que a tentativa também gera evento.

## Pré-condições

- Ficha `F1` do despacho com `equipe_id = EQ-07`.
- Quatro usuários: (1) `EquipeIntervencao` da `EQ-07`; (2) `EquipeIntervencao` da `EQ-09`; (3) `EquipeReceptora` com pré-admissão **ainda não enviada**; (4) `EquipeReceptora` com pré-admissão enviada.

## Dados de entrada

| Usuário | Esperado |
| --- | --- |
| 1 | acesso permitido |
| 2 | acesso negado |
| 3 | acesso negado |
| 4 | acesso permitido |

## Passos

1. Cada usuário tenta abrir a ficha `F1`.
2. Observa a tela e o resultado.
3. Consulta os eventos `PRONTUARIO_ACESSADO` publicados.

## Resultado esperado

- Usuários 1 e 4: ficha exibida; evento com `resultado = SUCESSO`.
- Usuários 2 e 3: mensagem de acesso negado, **nenhum dado clínico exibido**; evento com `resultado = NEGADO`.
- Quatro eventos no total, cada um com `ator_id`, `papel` e `entidade_id = F1`, sem conteúdo clínico.

## Resultado obtido

não executado

## Critério de aceite

Passa se os quatro desfechos acima ocorrem e cada tentativa gera exatamente um evento. Falha se algum acesso negado exibir dado ou ficar sem evento.

## Observações

Se G6 estiver indisponível, o evento fica `PENDENTE` e a leitura não é bloqueada (DS-G03-06).
