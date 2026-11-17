# ADR-004: Assinatura HMAC-SHA256 com secret por endpoint

## Status

Aceito. Decisão tomada na reunião técnica do Sistema de Webhooks de Notificação de Pedidos (`TRANSCRICAO.md`). Participaram da decisão: Sofia, que a propôs, Larissa, Diego e Bruno. A geração da secret pela plataforma veio de Marcos.

## Contexto

Os webhooks levam dados de pedidos para um endpoint fora da infraestrutura da empresa ([09:19] Sofia). O cliente precisa conseguir confirmar duas coisas em cada requisição: que ela veio mesmo da plataforma e que o payload não foi adulterado no caminho ([09:19] Sofia).

Secrets vazam: um cliente já expôs a sua em log de aplicação ([09:22] Diego).

## Decisão

Cada requisição de webhook é assinada com HMAC-SHA256 sobre o corpo do request ([09:20] Sofia; [09:22] Sofia), e a assinatura segue no header `X-Signature` ([09:44] Diego).

Cada endpoint de webhook cadastrado tem a sua própria secret ([09:21] Sofia), gerada pela plataforma e devolvida na criação do cadastro ([09:31] Marcos).

A secret é rotacionável pela API. Depois de uma rotação, a secret antiga continua válida por 24 horas, em paralelo com a nova, e então deixa de valer ([09:21] Sofia).

## Alternativas Consideradas

### Secret global da plataforma

Uma única secret para assinar os webhooks de todos os clientes. Descartada porque o vazamento dela comprometeria todos os endpoints de uma vez ([09:21] Sofia: “Senão se vaza uma, vaza tudo.”).

## Consequências

### Positivas

- O cliente verifica a origem e a integridade de cada requisição do lado dele ([09:20] Sofia).
- HMAC-SHA256 é padrão de mercado, com biblioteca disponível para os clientes ([09:20] Sofia).
- O vazamento de uma secret afeta apenas um endpoint ([09:21] Sofia).
- Depois de uma rotação, o cliente tem 24 horas, com a secret antiga ainda válida, para migrar os sistemas dele ([09:21] Sofia).

### Negativas

- A implementação de HMAC e da geração de secret passa por revisão de segurança antes do deploy, com pelo menos dois dias úteis reservados ([09:46] Sofia).
- A plataforma gera a secret de cada endpoint ([09:31] Marcos), guarda-a na configuração do webhook ([09:21] Bruno), oferece a rotação pela API e mantém a secret antiga válida por 24 horas depois dela ([09:21] Sofia).

### Trade-off

Aceitamos gerenciar uma secret por endpoint, com rotação e uma janela de 24 horas em que duas secrets convivem, para limitar o dano de um vazamento a um único endpoint e dar ao cliente tempo de migrar depois de uma rotação.

## Referências

- Transcrição: [09:19] Sofia; [09:20] Sofia; [09:21] Sofia; [09:21] Bruno; [09:22] Diego; [09:22] Sofia; [09:31] Marcos; [09:44] Diego; [09:46] Sofia.
- ADRs relacionados: [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md).
