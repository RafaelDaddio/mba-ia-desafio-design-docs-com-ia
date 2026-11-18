# RFC: Sistema de Webhooks de Notificação de Pedidos

## Metadados

| Campo | Valor |
| --- | --- |
| Autor | Rafael D'Addio |
| Status | Em revisão |
| Data | 17 de novembro de 2026 |
| Revisores | Larissa (Tech Lead), Marcos (Product Manager), Bruno (Engenheiro Pleno, time de Pedidos), Diego (Engenheiro Sênior, time de Plataforma), Sofia (Engenheira de Segurança) |
| Origem | Reunião técnica registrada em [TRANSCRICAO.md](../TRANSCRICAO.md) |

## Resumo executivo

Três clientes B2B pediram para ser avisados quando o status dos seus pedidos muda; hoje eles consultam o `GET /orders` de tempos em tempos ([09:00] Marcos). Este RFC propõe webhooks de saída, da plataforma para o cliente ([09:02] Marcos):

- a mudança de status grava um evento em uma outbox no MySQL, na mesma transação, quando há webhook do cliente do pedido inscrito naquele status;
- um worker em processo separado lê a outbox a cada 2 segundos e faz as chamadas HTTP;
- a entrega que falha é retentada com backoff exponencial e, esgotadas as retentativas, vai para uma DLQ com replay manual;
- cada requisição é assinada com HMAC-SHA256, com uma secret por endpoint;
- a garantia é at-least-once, e o cliente deduplica pelo header `X-Event-Id`.

Dois pontos não estão decididos: o rate limiting de envio e a forma de identificar o cliente no cadastro do webhook. Este RFC registra outros quatro pontos adiados ou fora do escopo desta feature. Todos estão na seção "Questões em aberto".

## Contexto e problema

Atlas Comercial, MaxDistribuição e Nova Cargo pediram formalmente notificação em tempo real das mudanças de status dos seus pedidos. Hoje eles consultam o `GET /orders` de tempos em tempos, o que deixa a integração “lenta e cara pra eles” ([09:00] Marcos). A Atlas sinalizou que pode migrar para um concorrente ([09:00] Marcos); o prazo e o risco de negócio estão no [PRD](PRD.md).

Para esses clientes, “qualquer coisa abaixo de 10 segundos já é "tempo real"” ([09:02] Marcos). O fluxo é só de saída: os clientes querem receber, não enviar ([09:02] Marcos).

No código, a mudança de status acontece em `OrderService.changeStatus` (`src/modules/orders/order.service.ts`), dentro de uma única transação, e a aplicação não tem fila nem mecanismo de evento. O [ADR-001](adrs/ADR-001-outbox-no-mysql.md) descreve esse ponto de partida.

## Proposta técnica

O caminho de um evento, do cadastro do webhook até o replay:

1. **Cadastro.** O cliente cadastra pela API um endpoint de webhook, com a URL e a lista de status que quer receber. A secret é gerada pela plataforma e devolvida na criação ([09:31] Marcos). A URL precisa ser https ([09:23] Sofia).
2. **Registro do evento.** Quando o status de um pedido muda e há webhook do cliente do pedido inscrito naquele status, o evento é gravado em uma outbox no MySQL, na mesma transação do `changeStatus` ([ADR-001](adrs/ADR-001-outbox-no-mysql.md)). O payload é gravado pronto, como snapshot daquele momento ([ADR-007](adrs/ADR-007-snapshot-do-payload-na-insercao.md)).
3. **Leitura.** Um único worker, em processo separado da API, lê a outbox em polling a cada 2 segundos ([ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md)).
4. **Envio.** O worker chama o endpoint do cliente por HTTP, com a requisição assinada em HMAC-SHA256 pela secret daquele endpoint ([ADR-004](adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md)). Cada evento leva um identificador único no header `X-Event-Id` ([ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)).
5. **Falha.** A entrega que falha tem 5 retentativas com backoff exponencial, em uma janela de quase 15 horas. Esgotadas as retentativas, o evento vai para uma DLQ em tabela separada ([ADR-003](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md)).
6. **Replay.** Um endpoint administrativo, restrito à role ADMIN, recoloca um evento da DLQ na outbox ([09:18] Diego; [09:36] Larissa).

Além do cadastro, a API permite editar, remover e listar os webhooks de um cliente ([09:33] Bruno), rotacionar a secret, com a antiga válida por 24 horas ([09:21] Sofia; [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md)), e consultar o histórico de entregas de um webhook ([09:34] Marcos).

O código novo entra como um módulo de webhooks nos moldes dos módulos existentes, reaproveitando as classes de erro, o logger, o middleware de erro, a validação e o controle de acesso que o projeto já tem ([ADR-006](adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md)).

Garantias e limites da proposta:

- **Entrega at-least-once.** O cliente pode receber o mesmo evento mais de uma vez ([09:24] Diego; [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)).
- **Latência.** O evento espera até 2 segundos pelo ciclo de leitura do worker, o que atende o limite de 10 segundos ([09:09] Diego; [09:10] Larissa; [ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md)).
- **Ordem.** Não há garantia de ordem global. A garantia vale por pedido, e só enquanto houver um único worker ([09:13] Larissa; [ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md)).

Contratos HTTP, tabelas, códigos de erro e fluxos detalhados ficam no [FDD](FDD.md).

## Alternativas consideradas

Alternativas de alcance arquitetural descartadas na reunião:

| Alternativa | Trade-off que levou ao descarte | Origem | Decisão |
| --- | --- | --- | --- |
| Envio síncrono dentro do service de orders | Um cliente lento travaria a mudança de status de outros pedidos, e com o cliente fora do ar desfazer a mudança de status não é opção. | [09:04] Bruno; [09:06] Diego | [ADR-001](adrs/ADR-001-outbox-no-mysql.md) |
| Redis Streams ou outra fila dedicada | Exigiria subir mais infraestrutura, o que foi considerado overengineering para um time pequeno. | [09:07] Larissa; [09:07] Diego | [ADR-001](adrs/ADR-001-outbox-no-mysql.md) |
| Trigger do banco no lugar do polling | Sugerida para tornar a entrega mais reativa, mas o MySQL não tem listener nativo: a trigger só executa SQL, e avisar o worker exigiria um improviso. | [09:09] Bruno; [09:09] Diego | [ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md) |
| Worker dentro do processo da API | O worker seria perdido a cada reinício da API. | [09:11] Diego | [ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md) |
| Garantia exactly-once | Exigiria coordenação dos dois lados e ficaria muito mais complexa. | [09:25] Diego | [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) |

As alternativas de alcance local estão nos ADRs: retry indefinido, três tentativas e marcação da falha na própria outbox ([ADR-003](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md)); secret global da plataforma ([ADR-004](adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md)); id auto incremental na outbox ([ADR-006](adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md)); montagem do payload só no envio ([ADR-007](adrs/ADR-007-snapshot-do-payload-na-insercao.md)).

## Questões em aberto

### Não decididas

| Questão | O que há até aqui | Origem |
| --- | --- | --- |
| Rate limiting de envio. Se 50 pedidos de um cliente mudam de status em um minuto, ele recebe 50 chamadas seguidas? | Diego acha que não entra no escopo agora: observar e implementar se virar problema. Sugeriu registrá-lo como ponto em aberto, e ficou como “observar e decidir depois”. | [09:38] Diego; [09:39] Diego; [09:39] Larissa |
| Como o cliente é identificado no cadastro do webhook: `customer_id` no body ou no path. | A reunião definiu que ele não vem do JWT e não escolheu entre as duas formas. | [09:32] Larissa |

### Adiadas ou fora do escopo desta feature

| Questão | O que há até aqui | Origem |
| --- | --- | --- |
| Vários workers em paralelo. | Perde-se a garantia de ordem. As saídas citadas foram particionar por pedido ou usar lock pessimista: “isso é problema do futuro, não agora”. | [09:12] Diego; [09:13] Diego |
| Alerta por email ao cliente quando o webhook dele falha. | Fica para a próxima fase. | [09:37] Larissa; [09:48] Larissa |
| Restringir as roles do CRUD de configuração, aberto a qualquer role autenticada nesta fase. | “Mais pra frente a gente pode endurecer.” | [09:37] Sofia |
| Arquivamento das linhas já entregues da outbox. | “depois de 30 dias ou assim, fora do escopo dessa feature” | [09:08] Diego |

## Impacto e riscos

### Impacto no sistema existente

- **Transação do `changeStatus`** (`src/modules/orders/order.service.ts`). Ganha a gravação na outbox; se ela falhar, a mudança de status é desfeita ([09:40] Bruno).
- **Processos.** A aplicação tem hoje um único entry point, `src/server.ts`. O worker pede um entry point e um script novos ([09:11] Larissa).
- **Banco.** Entram tabelas para a outbox ([09:06] Diego), a DLQ ([09:18] Diego) e a configuração dos webhooks ([09:21] Bruno; [09:21] Sofia), no MySQL que já existe ([09:07] Diego).
- **Código.** Entra um módulo de webhooks em `src/modules/` ([09:27] Bruno). O middleware de erro não muda ([09:29] Bruno).
- **Clientes.** Verificam a assinatura do lado deles ([09:20] Sofia) e precisam estar preparados para deduplicar pelo `X-Event-Id` ([09:24] Diego; [09:25] Diego). Marcos documenta a integração no portal ([09:40] Marcos).
- **Deploy.** A implementação de HMAC e da geração de secret passa por revisão de segurança antes do deploy ([09:46] Sofia; [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md)).

### Riscos

Esta seção traz os riscos de arquitetura e de segurança da proposta. Os riscos de negócio e de prazo ficam no [PRD](PRD.md), e os de implementação no [FDD](FDD.md).

| Risco | Origem | Tratamento |
| --- | --- | --- |
| Um cliente indisponível além da janela de retry deixa de receber o evento de forma automática. | [09:15] Diego; [09:17] Diego | Aceito pelo produto ([09:17] Marcos). O evento vai para a DLQ e pode ser reprocessado manualmente ([09:18] Diego). |
| O cliente recebe o mesmo evento mais de uma vez. | [09:24] Diego | Deduplicação pelo `X-Event-Id` ([09:25] Diego). Marcos se dispôs a documentar o comportamento no portal de desenvolvedor ([09:26] Marcos). |
| A garantia de ordem dos eventos de um pedido se perde se houver mais de um worker. | [09:12] Diego | Worker único por enquanto, com a limitação documentada ([09:13] Larissa). |
| Uma secret vaza do lado do cliente, o que já aconteceu uma vez. | [09:22] Diego | Secret por endpoint, para que o vazamento de uma não atinja as demais ([09:21] Sofia: “Senão se vaza uma, vaza tudo.”). A secret também é rotacionável ([09:21] Sofia); sobre a rotação, Diego disse: “Isso é importante. A gente já teve cliente que vazou secret em log de aplicação dele uma vez.” ([09:22] Diego). |
| A exigência de role ADMIN no replay pode ser contornada por autocadastro. | `src/modules/auth/auth.routes.ts`; `src/modules/auth/auth.schemas.ts` | Não tratado na reunião. Detalhado no [ADR-006](adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md). |
| Nenhum dado restringe um usuário aos webhooks do seu próprio cliente: `User` não tem vínculo com `Customer`, e o `customer_id` vem do body ou do path. | `prisma/schema.prisma`; [09:32] Larissa | Sobre restrição de acesso, a reunião tratou só de role: aceitou qualquer role autenticada nesta fase ([09:37] Sofia). |

## Decisões relacionadas

- [ADR-001: Outbox no MySQL para registrar eventos de webhook](adrs/ADR-001-outbox-no-mysql.md)
- [ADR-002: Worker em processo separado, lendo a outbox em polling](adrs/ADR-002-worker-em-processo-separado-com-polling.md)
- [ADR-003: Retry com backoff exponencial e DLQ em tabela separada](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md)
- [ADR-004: Assinatura HMAC-SHA256 com secret por endpoint](adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md)
- [ADR-005: Entrega at-least-once com deduplicação por X-Event-Id](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)
- [ADR-006: Reuso dos padrões existentes do projeto](adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md)
- [ADR-007: Snapshot do payload na inserção da outbox](adrs/ADR-007-snapshot-do-payload-na-insercao.md)
