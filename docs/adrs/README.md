# Architecture Decision Records

Este diretório reúne os ADRs do Sistema de Webhooks de Notificação de Pedidos. Cada arquivo registra uma decisão fechada na reunião técnica ([TRANSCRICAO.md](../../TRANSCRICAO.md)), no formato MADR: Status, Contexto, Decisão, Alternativas Consideradas e Consequências.

Os arquivos são nomeados como `ADR-NNN-titulo-em-kebab-case.md`.

## Índice

| ADR | Decisão | Status |
| --- | --- | --- |
| [ADR-001](ADR-001-outbox-no-mysql.md) | O evento de webhook é gravado em uma outbox no MySQL, na mesma transação da mudança de status. | Aceito |
| [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md) | Um único worker, em processo separado da API, lê a outbox em polling a cada 2 segundos. | Aceito |
| [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md) | Entregas com falha têm 5 retentativas com backoff exponencial e depois vão para uma DLQ em tabela separada. | Aceito |
| [ADR-004](ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) | Cada requisição é assinada com HMAC-SHA256, com uma secret por endpoint e rotação com 24 horas de convivência. | Aceito |
| [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) | A entrega é at-least-once, e o cliente deduplica pelo header `X-Event-Id`. | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-existentes-do-projeto.md) | O módulo de webhooks reaproveita os padrões que o projeto já tem. | Aceito |
| [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md) | O payload é gravado pronto na outbox, como snapshot do momento da mudança de status. | Aceito |
