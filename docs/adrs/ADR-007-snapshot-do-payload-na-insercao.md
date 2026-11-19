# ADR-007: Snapshot do payload na inserção da outbox

## Status

Aceito. Decisão tomada na reunião técnica do Sistema de Webhooks de Notificação de Pedidos (`TRANSCRICAO.md`), depois do resumo final e com Marcos e Sofia já fora da chamada. Participaram da decisão: Larissa, Diego e Bruno.

## Contexto

O evento fica na outbox ([ADR-001](ADR-001-outbox-no-mysql.md)) entre a mudança de status e o envio. O pedido pode mudar depois que o evento foi gravado ([09:52] Larissa).

A dúvida levantada foi o que a linha da outbox guarda: o payload já pronto, ou só a referência ao pedido, para montar o payload na hora do envio ([09:51] Bruno).

## Decisão

O payload é montado e gravado na outbox no momento da inserção, como um snapshot do pedido naquela mudança de status ([09:52] Larissa; [09:52] Diego; [09:52] Bruno).

## Alternativas Consideradas

### Guardar só o identificador do pedido e montar o payload no envio

Descartada porque, se o pedido mudar depois, o evento deixaria de refletir o estado de quando o status mudou ([09:52] Larissa: “Se o pedido mudar depois, o evento ainda reflete o estado de quando o status mudou.”).

## Consequências

### Positivas

- Cada evento descreve o pedido como ele estava na mudança de status que o gerou, mesmo que o pedido mude depois ([09:52] Larissa).

### Negativas

- A reunião não apontou custo para a escolha. O que a distingue da outra opção da pergunta de Bruno ([09:51] Bruno) é que cada linha da outbox guarda o payload inteiro, e não só o identificador do pedido ([09:52] Larissa).

### Trade-off

Aceitamos gravar o payload completo em cada linha da outbox para garantir que o evento descreve o pedido como ele estava quando o status mudou.

## Referências

- Transcrição: [09:51] Bruno; [09:52] Larissa; [09:52] Diego; [09:52] Bruno.
- ADRs relacionados: [ADR-001](ADR-001-outbox-no-mysql.md).
