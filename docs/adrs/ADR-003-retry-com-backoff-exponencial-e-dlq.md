# ADR-003: Retry com backoff exponencial e DLQ em tabela separada

## Status

Aceito. Decisão tomada na reunião técnica do Sistema de Webhooks de Notificação de Pedidos (`TRANSCRICAO.md`). Participaram da decisão: Larissa, Diego e Bruno, com aceite de Marcos para a janela de retry.

## Contexto

O sistema do cliente pode estar indisponível no momento em que o worker ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)) tenta entregar um evento ([09:14] Larissa). Já houve cliente com duas horas de indisponibilidade em manutenção planejada ([09:16] Diego). A política de reenvio precisa cobrir janelas desse tamanho sem deixar eventos presos para sempre.

A reunião fixou em 10 segundos o timeout de cada chamada: sem resposta nesse prazo, a entrega conta como falha e vai para retry ([09:42] Diego).

## Decisão

Uma entrega que falha é retentada com backoff exponencial: depois do envio inicial há 5 retentativas, com esperas de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas ([09:17] Diego; [09:17] Larissa).

Esgotadas as retentativas, o evento é considerado falha permanente e vai para uma DLQ: uma tabela separada da outbox, com o payload, o motivo da falha e o timestamp ([09:15] Diego; [09:18] Diego). O reprocessamento é manual, por um endpoint administrativo que recoloca o evento na outbox como pendente ([09:18] Diego; [09:19] Larissa).

**Leitura de “5 tentativas”.** A reunião fecha em “5 tentativas” ([09:17] Larissa). Este ADR adota a leitura de envio inicial mais 5 retentativas, porque só ela fecha com as contas feitas na própria reunião: três tentativas seriam “três vezes em 30 minutos” ([09:16] Diego), e os três primeiros intervalos somam 36 minutos; os cinco intervalos somam 14h36, o “quase 15 horas entre primeira falha e última tentativa” ([09:17] Diego).

## Alternativas Consideradas

### Retry indefinido com backoff

Descartado porque o evento ficaria pendurado para sempre se o cliente sumisse ([09:15] Diego).

### Três tentativas

Sugerida por ser mais agressiva ([09:16] Bruno). Descartada porque o sistema desistiria em cerca de 30 minutos, menos que as duas horas de indisponibilidade já vistas em um cliente ([09:16] Diego).

### Marcar a falha na própria outbox

Em vez de uma tabela separada, marcar o evento como "failed" na outbox ([09:17] Larissa). Descartada porque a tabela separada deixa a leitura da outbox principal mais limpa e serve de evidência para debug e reprocessamento ([09:18] Diego).

## Consequências

### Positivas

- A janela total, de quase 15 horas ([09:17] Diego), fica acima das duas horas de indisponibilidade já vistas em um cliente ([09:16] Diego) e foi aceita pelo produto ([09:17] Marcos).
- Nenhum evento fica em retry para sempre ([09:15] Diego).
- A DLQ preserva o payload e o motivo da falha, o que permite investigar e reprocessar ([09:18] Diego).

### Negativas

- Um cliente indisponível além da janela de quase 15 horas deixa de receber o evento de forma automática. A entrega passa a depender de um replay manual, restrito à role ADMIN ([09:36] Larissa).
- O aviso por email ao cliente quando o webhook dele falha ficou fora desta fase e vai para a próxima ([09:37] Larissa; [09:48] Larissa).

### Trade-off

Aceitamos desistir da entrega automática depois de quase 15 horas, e depender de replay manual a partir daí, para cobrir indisponibilidades longas sem manter eventos presos indefinidamente.

## Referências

- Transcrição: [09:14] Larissa; [09:15] Diego; [09:16] Bruno; [09:16] Diego; [09:17] Diego; [09:17] Marcos; [09:17] Larissa; [09:18] Diego; [09:19] Larissa; [09:36] Larissa; [09:37] Larissa; [09:42] Diego; [09:48] Larissa.
- ADRs relacionados: [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-existentes-do-projeto.md).
