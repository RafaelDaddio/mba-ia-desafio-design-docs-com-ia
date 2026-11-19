# ADR-001: Outbox no MySQL para registrar eventos de webhook

## Status

Aceito. Decisão tomada na reunião técnica do Sistema de Webhooks de Notificação de Pedidos (`TRANSCRICAO.md`). Participaram da decisão: Larissa, Diego e Bruno.

## Contexto

Três clientes B2B pediram para ser notificados quando o status dos seus pedidos muda ([09:00] Marcos). Hoje a mudança de status acontece em `OrderService.changeStatus` (`src/modules/orders/order.service.ts`), dentro de uma única transação que valida a transição, ajusta o estoque quando a transição exige, atualiza o pedido e grava o histórico. O sistema não tem fila, broker nem mecanismo de evento: nenhuma das dependências de `package.json` (Express, Prisma, Pino, pino-http, Zod, jsonwebtoken, bcrypt e uuid) é cliente de fila ou de mensageria.

A notificação precisa nascer dessa mudança de status sem que uma chamada HTTP a um sistema de terceiro entre na transação ([09:04] Bruno), e a reunião fixou a exigência: “Não pode ter caso de status mudar e evento não sair.” ([09:40] Bruno).

## Decisão

Quando o status de um pedido muda, o evento de webhook é gravado em uma tabela de outbox no MySQL já existente, dentro da mesma transação que atualiza o pedido e o histórico ([09:06] Diego; [09:08] Larissa), desde que algum webhook do cliente esteja inscrito naquele status ([09:34] Bruno). Se a gravação na outbox falhar, a transação inteira é desfeita ([09:40] Bruno).

O envio HTTP acontece depois, fora da transação, e é responsabilidade do worker descrito no [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md).

## Alternativas Consideradas

### Envio síncrono dentro do service de orders

Disparar a chamada HTTP no próprio `changeStatus`. Descartado por dois motivos ditos na reunião:

- Um cliente lento travaria a mudança de status de outros pedidos, porque a chamada ficaria dentro de uma transação que já é pesada ([09:04] Bruno).
- Não há o que fazer quando o cliente está fora do ar: desfazer a mudança de status não é opção ([09:04] Bruno: “o que a gente faz, dá rollback na mudança de status? Não dá.”).

Diego fechou o ponto: “Síncrono está fora de questão.” ([09:06] Diego).

### Redis Streams ou outra fila dedicada

Publicar o evento em uma fila externa. Descartado porque exigiria subir infraestrutura nova ([09:07] Larissa) para um time pequeno, o que foi considerado overengineering ([09:07] Diego).

## Consequências

### Positivas

- A mudança de status e o registro do evento são atômicos: se a transação confirma, o evento existe; se ela é desfeita, o evento some junto ([09:06] Diego).
- O filtro por status aplicado na inserção economiza linha na tabela ([09:34] Bruno).
- Nenhuma infraestrutura nova: “Outbox no MySQL existente resolve.” ([09:07] Diego).
- A chamada HTTP ao cliente fica fora da transação da mudança de status: quem dispara é um worker separado ([09:06] Diego).

### Negativas

- A transação do `changeStatus` ganha mais uma escrita, e uma falha nessa escrita desfaz a mudança de status ([09:40] Bruno: “Se a outbox falhar de inserir, rollback.”).
- A entrega não é imediata: o evento espera até 2 segundos pelo ciclo de leitura do worker ([09:10] Larissa; [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)).
- Bruno perguntou se o acúmulo de eventos na tabela deixaria o worker lento ([09:07] Bruno); Diego respondeu com a forma de leitura e os índices da tabela ([09:08] Diego). O arquivamento das linhas já entregues ficou fora do escopo desta feature ([09:08] Diego).

### Trade-off

Aceitamos uma entrega assíncrona, uma escrita a mais na transação do `changeStatus` e uma tabela sem arquivamento nesta fase para obter atomicidade entre a mudança de status e o registro do evento, sem infraestrutura nova.

## Referências

- Transcrição: [09:00] Marcos; [09:04] Bruno; [09:06] Diego; [09:07] Larissa; [09:07] Diego; [09:07] Bruno; [09:08] Diego; [09:08] Larissa; [09:10] Larissa; [09:34] Bruno; [09:40] Bruno.
- Código: `src/modules/orders/order.service.ts` (método `changeStatus`); `package.json` (dependências).
- ADRs relacionados: [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-existentes-do-projeto.md), [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md).
