# ADR-G03-03 — A ficha grava primeiro no tablet e sincroniza de forma idempotente

## Status

`aceito`

Data: 2026-11-26
Decisores: Luccas, Tulio, Geliton

## Contexto

O professor pergunta ao G3 o que fazer se a rede cair no local do atendimento (`contexto-do-sistema.md`).
O RNF-G03-02 exige 0 registros perdidos em queda de conexão e o RNF-G03-01 pede sincronização em até 5 s
(p95) após a reconexão. A arquitetura determina que o reenvio do mesmo `evento_id` não duplica e que a
ordem segue o timestamp do **fato**, não o do envio (`arquitetura-modular.md`, §6). A equipe usa um tablet
por viatura, em área com cobertura instável.

## Decisão

O tablet grava cada registro primeiro no armazenamento local, com `sincronizacao = PENDENTE`, e só depois
envia à central. O envio ocorre em ordem de `ocorrido_em`, cada registro com um `registro_id` ou `evento_id`
que torna o reenvio idempotente. Só após a confirmação da central o estado passa a `SINCRONIZADO`. A
interface nunca mostra um registro como salvo antes da gravação local.

## Alternativas consideradas

| Alternativa | Prós | Contras | Por que foi descartada |
| --- | --- | --- | --- |
| A. Só online, com fila de espera na tela | Sem armazenamento local | Uma queda perde o que está na tela | Contraria o RNF-G03-02. |
| B. Contingência em papel como único mecanismo | Funciona sem energia | A recepção redigita tudo, que é o problema que o módulo resolve | Não atende ao objetivo do projeto. |
| C. Offline-first com idempotência (escolhida) | Nenhuma perda; reenvio seguro | Exige armazenamento local protegido | — |
| D. Resolver conflitos por "último a gravar vence" | Simples | Sobrescreve dados clínicos | Contraria o ADR-G03-02 (registros são versionados, não sobrescritos). |

## Consequências

**Positivas**

- A equipe trabalha sem depender de rede, e o RNF-G03-02 passa a ser verificável por cenários de queda.
- O reenvio, inclusive o do C3 pendente, é seguro: a central reconhece o `evento_id` e não duplica.

**Negativas / custos**

- O dado clínico fica no dispositivo; a cifra em repouso (RNF-G03-03) e o bloqueio por inatividade (RNF-G03-08) viram obrigatórios.
- Premissa: um único tablet edita a ficha de um despacho, o que evita conflitos entre dispositivos. Se houver dois, será preciso outro ADR.
- Registros com `ocorrido_em` posterior ao encerramento do despacho ficam `sinalizado_para_revisao`, e o regulador precisa revisá-los.

**Impactos em outros grupos**

- G4: o C3 pode chegar atrasado, com o horário do fato original; G4 deve tratar o `evento_id` como idempotente (já previsto na arquitetura).

## Como verificar

- DS-G03-04: gravação local antes do envio, ordenação por `ocorrido_em` e ramo de idempotência.
- Dicionário de dados: `RegistroClinico.sincronizacao`, `ocorrido_em` e `registrado_em`.
- Casos de teste (M3): três cenários de queda (antes de salvar, durante o salvamento, durante a sincronização).
