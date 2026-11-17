# ADR-005: Entrega at-least-once com deduplicação por X-Event-Id

## Status

Aceito. Decisão tomada na reunião técnica do Sistema de Webhooks de Notificação de Pedidos (`TRANSCRICAO.md`). Participaram da decisão: Diego, que a propôs, Larissa, Sofia, Bruno e Marcos.

## Contexto

Ao tratar da garantia de entrega, a reunião reconheceu que o cliente pode receber o mesmo evento mais de uma vez e precisa estar preparado para isso ([09:24] Diego).

Era preciso decidir qual garantia a plataforma oferece e como o cliente distingue um evento novo de uma repetição ([09:25] Bruno).

## Decisão

A plataforma garante entrega at-least-once ([09:24] Diego; [09:26] Larissa).

Cada evento recebe um UUID no momento em que entra na outbox. Esse identificador é único por evento e segue no header `X-Event-Id` de cada envio. Cabe ao cliente deduplicar por ele ([09:25] Diego).

## Alternativas Consideradas

### Garantia exactly-once

Descartada porque exigiria coordenação entre a plataforma e o cliente e deixaria a solução muito mais complexa ([09:25] Diego).

## Consequências

### Positivas

- É o modelo que o mercado já usa: “Stripe faz assim, GitHub faz assim.” ([09:25] Diego).
- Na avaliação de Diego, “At-least-once com event_id resolve 99% dos casos.” ([09:25] Diego).
- Não exige coordenação entre os dois lados ([09:25] Diego).

### Negativas

- A responsabilidade de deduplicar fica com o cliente ([09:25] Sofia: “Isso joga responsabilidade pro cliente.”).
- Marcos se dispôs a documentar o comportamento com destaque no portal de desenvolvedor ([09:26] Marcos).

### Trade-off

Aceitamos que o cliente receba eventos repetidos e tenha de deduplicar para evitar a complexidade de coordenação que a garantia exactly-once exigiria.

## Referências

- Transcrição: [09:24] Diego; [09:25] Bruno; [09:25] Diego; [09:25] Sofia; [09:26] Marcos; [09:26] Larissa.
- ADRs relacionados: [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-004](ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md).
