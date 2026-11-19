# Tracker de rastreabilidade

Este documento liga cada item do [PRD](PRD.md), do [RFC](RFC.md), dos [ADRs](adrs/README.md) e do [FDD](FDD.md) à sua origem: uma fala da reunião registrada em [TRANSCRICAO.md](../TRANSCRICAO.md) ou um arquivo do repositório.

## Como ler

- **Uma linha por item.** O ID traz o documento, a categoria e o número do item. Quando o documento já numera o item, o número é o mesmo: `PRD-RF-07` é o RF-07 do PRD, e `FDD-P-12` é a Proposta P-12 do FDD.
- **Fonte.** `TRANSCRICAO` ou `CODIGO`. Quando o documento cita fala e arquivo para o mesmo item, a linha traz a origem do conteúdo resumido; a outra está no próprio documento.
- **Localização.** Minuto e falante, ou caminho do arquivo. Mais de uma referência vem separada por ponto e vírgula.
- **Item que aponta para um ADR.** O fluxo do RFC e os trade-offs do PRD remetem aos ADRs em vez de citar a fala. A linha traz a origem registrada no ADR correspondente.
- **Tipo "Proposta".** As propostas do FDD cobrem pontos que a reunião não definiu. A Localização é a fala ou o arquivo que tornou a definição necessária, e não uma decisão da reunião.
- **Decisões dos ADRs.** A decisão de cada ADR leva o ID do próprio ADR, como `ADR-002`. Uma segunda decisão ou uma leitura adotada no mesmo ADR leva o sufixo `DEC-02`.
- **Leituras e avaliações.** Onde um documento declara uma leitura ou uma avaliação própria, como a probabilidade e o impacto dos riscos do PRD, a linha traz as falas em que ela se apoia.

## Tabela

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-CTX-01 | `docs/PRD.md` | Contexto | O pedido percorre um ciclo de status, de PENDING a DELIVERED ou CANCELLED | CODIGO | `prisma/schema.prisma`; `src/modules/orders/order.status.ts` |
| PRD-CTX-02 | `docs/PRD.md` | Contexto | A plataforma não tem mecanismo de notificação: nenhuma dependência cumpre esse papel | CODIGO | `package.json` |
| PRD-CTX-03 | `docs/PRD.md` | Contexto | Atlas Comercial, MaxDistribuição e Nova Cargo pediram formalmente notificação em tempo real das mudanças de status | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-04 | `docs/PRD.md` | Contexto | Webhooks de saída: a plataforma chama um endpoint do cliente a cada mudança de status que ele escolheu acompanhar | TRANSCRICAO | [09:02] Marcos; [09:33] Marcos |
| PRD-PROB-01 | `docs/PRD.md` | Problema | Os clientes consultam o GET /orders de tempos em tempos, o que deixa a integração lenta e cara para eles | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-02 | `docs/PRD.md` | Problema | Os clientes pedem tempo real, e abaixo de 10 segundos já conta como tempo real | TRANSCRICAO | [09:02] Marcos |
| PRD-PROB-03 | `docs/PRD.md` | Motivação | A Atlas sinalizou que pode migrar para um concorrente e pediu a entrega para o fim de novembro | TRANSCRICAO | [09:00] Marcos; [09:45] Marcos |
| PRD-PUB-01 | `docs/PRD.md` | Público-alvo | Clientes B2B integradores, que recebem as notificações nos seus sistemas | TRANSCRICAO | [09:00] Marcos |
| PRD-PUB-02 | `docs/PRD.md` | Público-alvo | Usuários autenticados da API, que cadastram e administram os webhooks com o JWT do sistema | TRANSCRICAO | [09:32] Marcos; [09:32] Larissa |
| PRD-PUB-03 | `docs/PRD.md` | Público-alvo | Administradores, com a role ADMIN, que reprocessam eventos da DLQ | TRANSCRICAO | [09:36] Sofia |
| PRD-CEN-01 | `docs/PRD.md` | Cenário de uso | O cliente cadastra um webhook e passa a ser avisado, em vez de consultar a API | TRANSCRICAO | [09:00] Marcos; [09:31] Marcos |
| PRD-CEN-02 | `docs/PRD.md` | Cenário de uso | O cliente escolhe os status de cada webhook | TRANSCRICAO | [09:33] Marcos |
| PRD-CEN-03 | `docs/PRD.md` | Cenário de uso | Um cliente com vários cadastros identifica em cada envio qual deles foi usado | TRANSCRICAO | [09:44] Sofia |
| PRD-CEN-04 | `docs/PRD.md` | Cenário de uso | Um cliente em manutenção planejada, como a de duas horas que já aconteceu, tem as entregas retentadas | TRANSCRICAO | [09:16] Diego |
| PRD-CEN-05 | `docs/PRD.md` | Cenário de uso | O cliente consulta o histórico de entregas de um webhook | TRANSCRICAO | [09:34] Marcos |
| PRD-CEN-06 | `docs/PRD.md` | Cenário de uso | O cliente pede uma secret nova pela API e tem 24 horas para migrar os sistemas dele | TRANSCRICAO | [09:21] Sofia |
| PRD-CEN-07 | `docs/PRD.md` | Cenário de uso | Um administrador recoloca na outbox um evento que esgotou as retentativas | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| PRD-OBJ-01 | `docs/PRD.md` | Objetivo | Notificar em tempo real: menos de 10 segundos entre a mudança de status e o envio da notificação; os dois extremos da medição são leitura do PRD | TRANSCRICAO | [09:02] Marcos |
| PRD-FORA-01 | `docs/PRD.md` | Fora de escopo | Alerta por email quando o webhook do cliente falha: adiado para a próxima fase | TRANSCRICAO | [09:37] Larissa; [09:48] Larissa |
| PRD-FORA-02 | `docs/PRD.md` | Fora de escopo | Painel visual para o cliente: só endpoints, painel é projeto de outro time | TRANSCRICAO | [09:40] Larissa |
| PRD-FORA-03 | `docs/PRD.md` | Fora de escopo | Rate limiting de envio: observar e decidir depois | TRANSCRICAO | [09:39] Diego; [09:39] Larissa |
| PRD-FORA-04 | `docs/PRD.md` | Fora de escopo | Webhooks de entrada, do cliente para a plataforma: os clientes querem receber, não enviar | TRANSCRICAO | [09:02] Marcos |
| PRD-FORA-05 | `docs/PRD.md` | Fora de escopo | Garantia exactly-once: descartada | TRANSCRICAO | [09:25] Diego |
| PRD-FORA-06 | `docs/PRD.md` | Fora de escopo | Garantia de ordem global: sem garantia, e os clientes não pediram | TRANSCRICAO | [09:13] Larissa; [09:14] Marcos |
| PRD-FORA-07 | `docs/PRD.md` | Fora de escopo | Vários workers em paralelo: adiado | TRANSCRICAO | [09:13] Diego |
| PRD-FORA-08 | `docs/PRD.md` | Fora de escopo | Arquivamento das linhas entregues da outbox: fora do escopo desta feature | TRANSCRICAO | [09:08] Diego |
| PRD-FORA-09 | `docs/PRD.md` | Fora de escopo | Restrição de roles no CRUD de configuração: aberto a qualquer role autenticada nesta fase, e pode ser endurecido mais à frente | TRANSCRICAO | [09:37] Sofia |
| PRD-RF-01 | `docs/PRD.md` | Requisito Funcional | Cadastrar um webhook pela API, informando a URL e a lista de status a receber | TRANSCRICAO | [09:31] Marcos |
| PRD-RF-02 | `docs/PRD.md` | Requisito Funcional | Gerar a secret do webhook na plataforma e devolvê-la na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-RF-03 | `docs/PRD.md` | Requisito Funcional | Editar um webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-04 | `docs/PRD.md` | Requisito Funcional | Remover um webhook | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-05 | `docs/PRD.md` | Requisito Funcional | Listar os webhooks de um cliente | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-06 | `docs/PRD.md` | Requisito Funcional | Escolher, por webhook, quais status geram notificação | TRANSCRICAO | [09:33] Bruno; [09:33] Marcos |
| PRD-RF-07 | `docs/PRD.md` | Requisito Funcional | Notificar o cliente quando o status de um pedido dele muda para um status em que há webhook inscrito | TRANSCRICAO | [09:00] Marcos; [09:34] Bruno |
| PRD-RF-08 | `docs/PRD.md` | Requisito Funcional | Enviar na notificação os dados do evento e do pedido, sem os itens do pedido | TRANSCRICAO | [09:43] Diego |
| PRD-RF-09 | `docs/PRD.md` | Requisito Funcional | Identificar cada evento com um identificador único, para o cliente reconhecer repetições | TRANSCRICAO | [09:25] Diego |
| PRD-RF-10 | `docs/PRD.md` | Requisito Funcional | Informar em cada envio qual webhook cadastrado o originou | TRANSCRICAO | [09:44] Sofia; [09:45] Diego |
| PRD-RF-11 | `docs/PRD.md` | Requisito Funcional | Rotacionar a secret pela API, mantendo a antiga válida por 24 horas | TRANSCRICAO | [09:21] Sofia |
| PRD-RF-12 | `docs/PRD.md` | Requisito Funcional | Consultar o histórico de entregas de um webhook, com sucesso ou falha, payload, response e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| PRD-RF-13 | `docs/PRD.md` | Requisito Funcional | Retentar automaticamente as entregas que falham e, esgotadas as retentativas, guardar o evento em uma DLQ | TRANSCRICAO | [09:15] Diego; [09:18] Diego |
| PRD-RF-14 | `docs/PRD.md` | Requisito Funcional | Reprocessar manualmente um evento da DLQ, por um endpoint restrito à role ADMIN | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| PRD-RF-15 | `docs/PRD.md` | Requisito Funcional | Registrar quem executou cada replay, para auditoria | TRANSCRICAO | [09:36] Sofia |
| PRD-RF-16 | `docs/PRD.md` | Requisito Funcional | Informar em cada envio o horário do envio, para o cliente poder detectar replay attack se quiser | TRANSCRICAO | [09:44] Diego |
| PRD-QA-01 | `docs/PRD.md` | Questão em aberto | O identificador do cliente não vem do JWT: vai no body ou no path, e a reunião não escolheu entre os dois | TRANSCRICAO | [09:32] Larissa |
| PRD-LIM-01 | `docs/PRD.md` | Limitação | Um webhook inscrito em PENDING nunca dispara: nenhuma transição leva a esse status, e a criação do pedido não passa pela mudança de status | CODIGO | `src/modules/orders/order.status.ts`; `src/modules/orders/order.service.ts` |
| PRD-RNF-01 | `docs/PRD.md` | Requisito Não Funcional | Latência: a notificação sai em menos de 10 segundos, com a outbox lida a cada 2 segundos | TRANSCRICAO | [09:02] Marcos; [09:09] Diego |
| PRD-RNF-02 | `docs/PRD.md` | Requisito Não Funcional | Consistência: o evento é registrado na mesma transação da mudança de status | TRANSCRICAO | [09:06] Diego; [09:40] Bruno |
| PRD-RNF-03 | `docs/PRD.md` | Requisito Não Funcional | Entrega at-least-once: o cliente pode receber o mesmo evento mais de uma vez | TRANSCRICAO | [09:24] Diego |
| PRD-RNF-04 | `docs/PRD.md` | Requisito Não Funcional | Autenticidade e integridade: cada requisição é assinada com HMAC-SHA256, com uma secret por endpoint | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| PRD-RNF-05 | `docs/PRD.md` | Requisito Não Funcional | Transporte: a URL do webhook precisa ser https, e http é recusado | TRANSCRICAO | [09:23] Sofia |
| PRD-RNF-06 | `docs/PRD.md` | Requisito Não Funcional | Tamanho: o payload tem limite de 64 KB, com erro caso ultrapasse | TRANSCRICAO | [09:24] Larissa |
| PRD-RNF-07 | `docs/PRD.md` | Requisito Não Funcional | Timeout: 10 segundos por chamada; sem resposta, a entrega conta como falha | TRANSCRICAO | [09:42] Diego |
| PRD-RNF-08 | `docs/PRD.md` | Requisito Não Funcional | Resiliência: envio inicial mais 5 retentativas com backoff exponencial, em uma janela de quase 15 horas | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| PRD-RNF-09 | `docs/PRD.md` | Requisito Não Funcional | Ordem: garantida por pedido, enquanto houver um único worker | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| PRD-RNF-10 | `docs/PRD.md` | Requisito Não Funcional | Acesso: nesta fase, o CRUD de configuração aceita qualquer role autenticada, e o replay exige ADMIN | TRANSCRICAO | [09:36] Sofia; [09:37] Sofia |
| PRD-RNF-11 | `docs/PRD.md` | Requisito Não Funcional | Infraestrutura: mesmo banco e mesma stack; a outbox usa o MySQL existente, sem subir Redis | TRANSCRICAO | [09:07] Diego; [09:11] Diego |
| PRD-RNF-12 | `docs/PRD.md` | Requisito Não Funcional | Código: o módulo segue os padrões do projeto | TRANSCRICAO | [09:30] Larissa |
| PRD-TO-01 | `docs/PRD.md` | Trade-off | Outbox no MySQL: aceita-se entrega assíncrona e mais uma escrita na transação da mudança de status | TRANSCRICAO | [09:10] Larissa; [09:40] Bruno |
| PRD-TO-02 | `docs/PRD.md` | Trade-off | Worker em processo separado, com polling: aceita-se até 2 segundos de espera e um único worker por enquanto | TRANSCRICAO | [09:10] Larissa; [09:13] Larissa |
| PRD-TO-03 | `docs/PRD.md` | Trade-off | Retry com backoff e DLQ: aceita-se desistir da entrega automática depois de quase 15 horas | TRANSCRICAO | [09:17] Diego; [09:17] Marcos |
| PRD-TO-04 | `docs/PRD.md` | Trade-off | HMAC-SHA256 com secret por endpoint: aceita-se gerenciar uma secret por endpoint, com rotação e janela de 24 horas | TRANSCRICAO | [09:21] Sofia |
| PRD-TO-05 | `docs/PRD.md` | Trade-off | At-least-once com X-Event-Id: o cliente pode receber repetições e precisa deduplicar | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| PRD-TO-06 | `docs/PRD.md` | Trade-off | Reuso dos padrões do projeto: aceitam-se as limitações dos componentes como estão hoje | CODIGO | `src/shared/errors/http-errors.ts`; `src/shared/logger/index.ts` |
| PRD-TO-07 | `docs/PRD.md` | Trade-off | Snapshot do payload na inserção: aceita-se gravar o payload completo em cada linha da outbox | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa |
| PRD-DEP-01 | `docs/PRD.md` | Dependência | Revisão de segurança do código de HMAC e de geração de secret antes do deploy, com pelo menos dois dias úteis reservados | TRANSCRICAO | [09:46] Sofia; [09:49] Sofia |
| PRD-DEP-02 | `docs/PRD.md` | Dependência | Documentação da integração no portal para os clientes, por Marcos | TRANSCRICAO | [09:40] Marcos |
| PRD-DEP-03 | `docs/PRD.md` | Dependência | Confirmação do prazo com a Atlas e aviso aos clientes, por Marcos | TRANSCRICAO | [09:47] Marcos; [09:49] Marcos |
| PRD-DEP-04 | `docs/PRD.md` | Dependência | Sessão de revisão do documento de design antes de começar a codar | TRANSCRICAO | [09:50] Larissa |
| PRD-DEP-05 | `docs/PRD.md` | Dependência | O cliente verifica a assinatura do lado dele e precisa estar preparado para receber eventos repetidos | TRANSCRICAO | [09:20] Sofia; [09:24] Diego |
| PRD-DEP-06 | `docs/PRD.md` | Dependência | MySQL e base de código existentes | TRANSCRICAO | [09:07] Diego; [09:30] Larissa |
| PRD-PRAZO-01 | `docs/PRD.md` | Prazo | A Atlas pediu a entrega para o fim de novembro | TRANSCRICAO | [09:45] Marcos |
| PRD-PRAZO-02 | `docs/PRD.md` | Prazo | Estimativa de três sprints, dividida por parte, com a revisão de segurança incluída no fim | TRANSCRICAO | [09:46] Larissa; [09:47] Larissa |
| PRD-RISCO-01 | `docs/PRD.md` | Risco | A Atlas migra para um concorrente se a feature não for entregue no prazo que pediu; Marcos confirma o prazo e atualiza os clientes | TRANSCRICAO | [09:00] Marcos; [09:45] Marcos; [09:47] Marcos; [09:49] Marcos |
| PRD-RISCO-02 | `docs/PRD.md` | Risco | A estimativa de três sprints, dada como chute, não se confirma | TRANSCRICAO | [09:46] Larissa |
| PRD-RISCO-03 | `docs/PRD.md` | Risco | A revisão de segurança, pré-condição do deploy, não acontece a tempo; reservar pelo menos dois dias úteis e agendar com Sofia | TRANSCRICAO | [09:46] Sofia; [09:47] Larissa; [09:49] Sofia |
| PRD-RISCO-04 | `docs/PRD.md` | Risco | O cliente não é avisado quando o webhook dele está falhando, porque o alerta por email ficou para a próxima fase | TRANSCRICAO | [09:16] Diego; [09:17] Diego; [09:37] Marcos; [09:37] Larissa; [09:48] Larissa |
| PRD-CA-01 | `docs/PRD.md` | Critério de aceitação | Depois da mudança de status, o envio ao webhook do cliente inscrito naquele status sai em menos de 10 segundos | TRANSCRICAO | [09:02] Marcos; [09:34] Bruno |
| PRD-CA-02 | `docs/PRD.md` | Critério de aceitação | O cliente recebe só os status que escolheu para aquele webhook | TRANSCRICAO | [09:33] Marcos |
| PRD-CA-03 | `docs/PRD.md` | Critério de aceitação | O cliente consegue validar que a requisição veio da plataforma e que o payload não foi adulterado | TRANSCRICAO | [09:19] Sofia |
| PRD-CA-04 | `docs/PRD.md` | Critério de aceitação | O cliente consegue reconhecer um evento repetido pelo identificador do evento | TRANSCRICAO | [09:25] Diego |
| PRD-CA-05 | `docs/PRD.md` | Critério de aceitação | O cliente consegue rotacionar a secret, e a antiga continua válida por 24 horas | TRANSCRICAO | [09:21] Sofia |
| PRD-CA-06 | `docs/PRD.md` | Critério de aceitação | O cliente consegue consultar o histórico de entregas de um webhook | TRANSCRICAO | [09:34] Marcos |
| PRD-CA-07 | `docs/PRD.md` | Critério de aceitação | Um evento que esgotou as retentativas pode ser reprocessado por um administrador, e o replay registra quem o executou | TRANSCRICAO | [09:18] Diego; [09:36] Sofia |
| PRD-CA-08 | `docs/PRD.md` | Critério de aceitação | O código de HMAC e de geração de secret passou pela revisão de segurança antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-TESTE-01 | `docs/PRD.md` | Estratégia de testes | Testes ponta a ponta, estimados junto com a integração no service de pedidos | TRANSCRICAO | [09:46] Larissa |
| PRD-TESTE-02 | `docs/PRD.md` | Estratégia de testes | Proposta do documento: testar os endpoints novos com Vitest e supertest contra o MySQL real, como os testes de pedidos | CODIGO | `tests/orders.test.ts`; `tests/setup.ts`; `package.json` |
| PRD-TESTE-04 | `docs/PRD.md` | Estratégia de testes | Revisão de segurança do código de HMAC e de geração de secret, por Sofia, antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-TESTE-05 | `docs/PRD.md` | Estratégia de testes | Revisão do documento de design por Larissa, Bruno e Diego antes de começar a codar | TRANSCRICAO | [09:50] Larissa |
| RFC-CTX-01 | `docs/RFC.md` | Contexto | Três clientes B2B pediram notificação em tempo real; hoje consultam o GET /orders de tempos em tempos | TRANSCRICAO | [09:00] Marcos |
| RFC-CTX-02 | `docs/RFC.md` | Restrição | Para os clientes, abaixo de 10 segundos já é tempo real | TRANSCRICAO | [09:02] Marcos |
| RFC-CTX-03 | `docs/RFC.md` | Restrição | O fluxo é só de saída: os clientes querem receber, não enviar | TRANSCRICAO | [09:02] Marcos |
| RFC-CTX-04 | `docs/RFC.md` | Contexto | A mudança de status acontece em OrderService.changeStatus, dentro de uma única transação, e a aplicação não tem fila nem mecanismo de evento | CODIGO | `src/modules/orders/order.service.ts`; `package.json` |
| RFC-CTX-05 | `docs/RFC.md` | Contexto | A Atlas sinalizou que pode migrar para um concorrente | TRANSCRICAO | [09:00] Marcos |
| RFC-PROP-01 | `docs/RFC.md` | Proposta técnica | Cadastro pela API com URL e lista de status; secret gerada pela plataforma e devolvida na criação; URL https | TRANSCRICAO | [09:31] Marcos; [09:23] Sofia |
| RFC-PROP-02 | `docs/RFC.md` | Proposta técnica | O evento é gravado em uma outbox no MySQL, na transação do changeStatus, quando há webhook do cliente do pedido inscrito naquele status; o payload é gravado como snapshot | TRANSCRICAO | [09:06] Diego; [09:08] Larissa; [09:34] Bruno; [09:52] Larissa |
| RFC-PROP-03 | `docs/RFC.md` | Proposta técnica | Um único worker, em processo separado da API, lê a outbox em polling a cada 2 segundos | TRANSCRICAO | [09:09] Diego; [09:11] Diego; [09:12] Diego |
| RFC-PROP-04 | `docs/RFC.md` | Proposta técnica | O worker chama o endpoint do cliente com a requisição assinada em HMAC-SHA256 pela secret do endpoint, e cada evento leva um identificador único no X-Event-Id | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:25] Diego |
| RFC-PROP-05 | `docs/RFC.md` | Proposta técnica | A entrega que falha tem 5 retentativas com backoff exponencial, em quase 15 horas; depois, o evento vai para uma DLQ em tabela separada | TRANSCRICAO | [09:17] Diego; [09:17] Larissa; [09:18] Diego |
| RFC-PROP-06 | `docs/RFC.md` | Proposta técnica | Um endpoint administrativo, restrito à role ADMIN, recoloca um evento da DLQ na outbox | TRANSCRICAO | [09:18] Diego; [09:36] Larissa |
| RFC-PROP-07 | `docs/RFC.md` | Proposta técnica | A API permite editar, remover e listar os webhooks de um cliente | TRANSCRICAO | [09:33] Bruno |
| RFC-PROP-08 | `docs/RFC.md` | Proposta técnica | A API permite rotacionar a secret, com a antiga válida por 24 horas | TRANSCRICAO | [09:21] Sofia |
| RFC-PROP-09 | `docs/RFC.md` | Proposta técnica | A API permite consultar o histórico de entregas de um webhook | TRANSCRICAO | [09:34] Marcos |
| RFC-PROP-10 | `docs/RFC.md` | Proposta técnica | O código novo entra como um módulo de webhooks nos moldes dos existentes, reaproveitando erros, logger, middleware de erro, validação e controle de acesso | TRANSCRICAO | [09:27] Bruno; [09:30] Larissa; [09:36] Larissa |
| RFC-GAR-01 | `docs/RFC.md` | Garantia | Entrega at-least-once: o cliente pode receber o mesmo evento mais de uma vez | TRANSCRICAO | [09:24] Diego |
| RFC-GAR-02 | `docs/RFC.md` | Garantia | Latência: o evento espera até 2 segundos pelo ciclo de leitura, o que atende o limite de 10 segundos | TRANSCRICAO | [09:09] Diego; [09:10] Larissa |
| RFC-GAR-03 | `docs/RFC.md` | Garantia | Ordem: sem garantia global; vale por pedido, e só enquanto houver um único worker | TRANSCRICAO | [09:13] Larissa |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa descartada | Envio síncrono dentro do service de orders: um cliente lento travaria outros pedidos, e desfazer a mudança de status não é opção | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa descartada | Redis Streams ou outra fila dedicada: exigiria mais infraestrutura, overengineering para um time pequeno | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa descartada | Trigger do banco no lugar do polling: o MySQL não tem listener nativo, e avisar o worker exigiria um improviso | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa descartada | Worker dentro do processo da API: seria perdido a cada reinício da API | TRANSCRICAO | [09:11] Diego |
| RFC-ALT-05 | `docs/RFC.md` | Alternativa descartada | Garantia exactly-once: exigiria coordenação dos dois lados e ficaria muito mais complexa | TRANSCRICAO | [09:25] Diego |
| RFC-QA-01 | `docs/RFC.md` | Questão em aberto | Rate limiting de envio: não decidido; observar e decidir depois | TRANSCRICAO | [09:38] Diego; [09:39] Diego; [09:39] Larissa |
| RFC-QA-02 | `docs/RFC.md` | Questão em aberto | Identificação do cliente no cadastro: customer_id no body ou no path, sem escolha na reunião | TRANSCRICAO | [09:32] Larissa |
| RFC-QA-03 | `docs/RFC.md` | Item adiado | Vários workers em paralelo: perde-se a garantia de ordem; problema do futuro | TRANSCRICAO | [09:12] Diego; [09:13] Diego |
| RFC-QA-04 | `docs/RFC.md` | Item adiado | Alerta por email ao cliente quando o webhook dele falha: próxima fase | TRANSCRICAO | [09:37] Larissa; [09:48] Larissa |
| RFC-QA-05 | `docs/RFC.md` | Fora de escopo | Restringir as roles do CRUD de configuração, aberto a qualquer role autenticada nesta fase | TRANSCRICAO | [09:37] Sofia |
| RFC-QA-06 | `docs/RFC.md` | Item adiado | Arquivamento das linhas já entregues da outbox: fora do escopo desta feature | TRANSCRICAO | [09:08] Diego |
| RFC-QA-07 | `docs/RFC.md` | Fora de escopo | Painel visual para o cliente: só endpoints nesta feature; painel é projeto separado do time de frontend | TRANSCRICAO | [09:40] Larissa |
| RFC-IMP-01 | `docs/RFC.md` | Impacto | A transação do changeStatus ganha a gravação na outbox; se ela falhar, a mudança de status é desfeita | TRANSCRICAO | [09:40] Bruno |
| RFC-IMP-02 | `docs/RFC.md` | Impacto | O worker pede um entry point e um script novos, além do src/server.ts que existe hoje | TRANSCRICAO | [09:11] Larissa |
| RFC-IMP-03 | `docs/RFC.md` | Impacto | Entram tabelas para a outbox, a DLQ e a configuração dos webhooks, no MySQL que já existe; o histórico de entregas também precisa ser guardado | TRANSCRICAO | [09:06] Diego; [09:07] Diego; [09:18] Diego; [09:21] Bruno; [09:21] Sofia; [09:34] Marcos |
| RFC-IMP-04 | `docs/RFC.md` | Impacto | Entra um módulo de webhooks em src/modules/, e o middleware de erro não muda | TRANSCRICAO | [09:27] Bruno; [09:29] Bruno |
| RFC-IMP-05 | `docs/RFC.md` | Impacto | Os clientes verificam a assinatura e precisam estar preparados para deduplicar; Marcos documenta a integração no portal | TRANSCRICAO | [09:20] Sofia; [09:24] Diego; [09:25] Diego; [09:40] Marcos |
| RFC-IMP-06 | `docs/RFC.md` | Impacto | A implementação de HMAC e da geração de secret passa por revisão de segurança antes do deploy | TRANSCRICAO | [09:46] Sofia |
| RFC-RISCO-01 | `docs/RFC.md` | Risco | Um cliente indisponível além da janela de retry deixa de receber o evento de forma automática; aceito pelo produto, com DLQ e replay manual | TRANSCRICAO | [09:15] Diego; [09:17] Diego; [09:17] Marcos; [09:18] Diego |
| RFC-RISCO-02 | `docs/RFC.md` | Risco | O cliente recebe o mesmo evento mais de uma vez; deduplicação pelo X-Event-Id | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Marcos |
| RFC-RISCO-03 | `docs/RFC.md` | Risco | A garantia de ordem dos eventos de um pedido se perde com mais de um worker; worker único por enquanto | TRANSCRICAO | [09:12] Diego; [09:13] Larissa |
| RFC-RISCO-04 | `docs/RFC.md` | Risco | Uma secret vaza do lado do cliente, o que já aconteceu uma vez; secret por endpoint e rotacionável | TRANSCRICAO | [09:22] Diego; [09:21] Sofia |
| RFC-RISCO-05 | `docs/RFC.md` | Risco | A exigência de role ADMIN no replay pode ser contornada por autocadastro | CODIGO | `src/modules/auth/auth.routes.ts`; `src/modules/auth/auth.schemas.ts` |
| RFC-RISCO-06 | `docs/RFC.md` | Risco | Nenhum dado restringe um usuário aos webhooks do seu próprio cliente: User não tem vínculo com Customer | CODIGO | `prisma/schema.prisma` |
| ADR-001 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | O evento é gravado em uma outbox no MySQL, na transação que atualiza o pedido e o histórico, se algum webhook do cliente está inscrito naquele status; se a gravação falha, a transação é desfeita | TRANSCRICAO | [09:06] Diego; [09:08] Larissa; [09:34] Bruno; [09:40] Bruno |
| ADR-001-CTX-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Contexto | A mudança de status acontece em uma única transação no changeStatus, e nenhuma dependência é cliente de fila ou de mensageria | CODIGO | `src/modules/orders/order.service.ts`; `package.json` |
| ADR-001-CTX-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Restrição | A chamada HTTP a terceiro não entra na transação, e não pode haver status que muda sem evento | TRANSCRICAO | [09:04] Bruno; [09:40] Bruno |
| ADR-001-CTX-03 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Contexto | Três clientes B2B pediram para ser notificados quando o status dos seus pedidos muda | TRANSCRICAO | [09:00] Marcos |
| ADR-001-ALT-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa descartada | Envio síncrono dentro do service de orders: cliente lento trava a transação, e não dá para desfazer a mudança de status | TRANSCRICAO | [09:04] Bruno; [09:06] Diego |
| ADR-001-ALT-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa descartada | Redis Streams ou outra fila dedicada: infraestrutura nova, overengineering para um time pequeno | TRANSCRICAO | [09:07] Larissa; [09:07] Diego |
| ADR-001-POS-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência positiva | A mudança de status e o registro do evento são atômicos | TRANSCRICAO | [09:06] Diego |
| ADR-001-POS-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência positiva | O filtro por status aplicado na inserção economiza linha na tabela | TRANSCRICAO | [09:34] Bruno |
| ADR-001-POS-03 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência positiva | Nenhuma infraestrutura nova: a outbox usa o MySQL existente | TRANSCRICAO | [09:07] Diego |
| ADR-001-POS-04 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência positiva | A chamada HTTP fica fora da transação: quem dispara é um worker separado | TRANSCRICAO | [09:06] Diego |
| ADR-001-NEG-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência negativa | A transação ganha mais uma escrita, e uma falha nela desfaz a mudança de status | TRANSCRICAO | [09:40] Bruno |
| ADR-001-NEG-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência negativa | A entrega não é imediata: o evento espera até 2 segundos pelo ciclo de leitura do worker | TRANSCRICAO | [09:10] Larissa |
| ADR-001-NEG-03 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Consequência negativa | Bruno perguntou se o acúmulo de eventos deixaria o worker lento; Diego respondeu com a leitura e os índices, e o arquivamento ficou fora do escopo | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| ADR-001-TO | `docs/adrs/ADR-001-outbox-no-mysql.md` | Trade-off | Entrega assíncrona, uma escrita a mais e tabela sem arquivamento, em troca de atomicidade sem infraestrutura nova | TRANSCRICAO | [09:06] Diego; [09:07] Diego; [09:08] Diego; [09:10] Larissa; [09:40] Bruno |
| ADR-002 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Decisão | Um único worker, em processo separado da API, lê a outbox em polling a cada 2 segundos, com o mesmo banco e a mesma stack | TRANSCRICAO | [09:09] Diego; [09:10] Larissa; [09:11] Diego; [09:11] Larissa; [09:12] Diego |
| ADR-002-DEC-02 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Decisão | O worker abre a sua própria instância do PrismaClient, com a mesma DATABASE_URL | TRANSCRICAO | [09:30] Bruno |
| ADR-002-CTX-01 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Contexto | O MySQL não oferece aviso nativo a processos externos quando uma linha é inserida | TRANSCRICAO | [09:09] Diego |
| ADR-002-CTX-02 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Contexto | A aplicação tem um único entry point, e os scripts dev e start iniciam só esse processo | CODIGO | `src/server.ts`; `package.json` |
| ADR-002-CTX-03 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Contexto | Para os clientes, qualquer atraso abaixo de 10 segundos já conta como tempo real | TRANSCRICAO | [09:02] Marcos |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Alternativa descartada | Trigger do banco para reagir na hora: a trigger só executa SQL e não avisa um processo externo | TRANSCRICAO | [09:09] Bruno; [09:09] Diego |
| ADR-002-ALT-02 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Alternativa descartada | Worker dentro do processo da API: cairia junto a cada reinício da API | TRANSCRICAO | [09:11] Diego |
| ADR-002-POS-01 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Consequência positiva | O intervalo de 2 segundos atende o limite de 10 segundos e foi aceito pelo produto | TRANSCRICAO | [09:09] Diego; [09:10] Marcos |
| ADR-002-POS-02 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Consequência positiva | O worker não é perdido quando a API reinicia | TRANSCRICAO | [09:11] Diego |
| ADR-002-POS-03 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Consequência positiva | Com um único worker processando por ordem de criação, o cliente recebe em ordem os eventos de um mesmo pedido | TRANSCRICAO | [09:12] Diego |
| ADR-002-NEG-01 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Consequência negativa | Todo evento espera até 2 segundos pelo próximo ciclo de leitura | TRANSCRICAO | [09:10] Larissa |
| ADR-002-NEG-02 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Consequência negativa | Não há garantia de ordem global, e os clientes não a pediram | TRANSCRICAO | [09:13] Larissa; [09:14] Marcos |
| ADR-002-NEG-03 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Consequência negativa | Escalar para vários workers quebra a garantia de ordem; o assunto foi adiado | TRANSCRICAO | [09:12] Diego; [09:13] Diego |
| ADR-002-NEG-04 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Consequência negativa | O worker depende de um entry point e de um script npm run worker que ainda não existem | TRANSCRICAO | [09:11] Larissa |
| ADR-002-TO | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Trade-off | Até 2 segundos de atraso e um único worker, sem ordem global, em troca de cumprir o limite de 10 segundos com o mesmo banco e a mesma stack | TRANSCRICAO | [09:10] Larissa; [09:11] Diego; [09:13] Larissa |
| ADR-003 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Decisão | Envio inicial mais 5 retentativas, com esperas de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas; depois, DLQ em tabela separada e replay manual por endpoint administrativo | TRANSCRICAO | [09:15] Diego; [09:17] Diego; [09:17] Larissa; [09:18] Diego; [09:19] Larissa |
| ADR-003-DEC-02 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Leitura adotada | "5 tentativas" é lido como envio inicial mais 5 retentativas, pela conta de quase 15 horas feita na reunião | TRANSCRICAO | [09:16] Diego; [09:17] Diego; [09:17] Larissa |
| ADR-003-CTX-01 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Contexto | O sistema do cliente pode estar indisponível; já houve cliente com duas horas de manutenção planejada | TRANSCRICAO | [09:14] Larissa; [09:16] Diego |
| ADR-003-CTX-02 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Restrição | Timeout de 10 segundos por chamada: sem resposta, a entrega conta como falha e vai para retry | TRANSCRICAO | [09:42] Diego |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa descartada | Retry indefinido com backoff: o evento ficaria pendurado para sempre se o cliente sumisse | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa descartada | Três tentativas: o sistema desistiria em cerca de 30 minutos, menos que as duas horas já vistas | TRANSCRICAO | [09:16] Bruno; [09:16] Diego |
| ADR-003-ALT-03 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa descartada | Marcar a falha na própria outbox: a tabela separada deixa a leitura mais limpa e serve de evidência | TRANSCRICAO | [09:17] Larissa; [09:18] Diego |
| ADR-003-POS-01 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Consequência positiva | A janela de quase 15 horas fica acima das duas horas de indisponibilidade já vistas e foi aceita pelo produto | TRANSCRICAO | [09:16] Diego; [09:17] Diego; [09:17] Marcos |
| ADR-003-POS-02 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Consequência positiva | Nenhum evento fica em retry para sempre | TRANSCRICAO | [09:15] Diego |
| ADR-003-POS-03 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Consequência positiva | A DLQ preserva o payload e o motivo da falha, o que permite investigar e reprocessar | TRANSCRICAO | [09:18] Diego |
| ADR-003-NEG-01 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Consequência negativa | Além da janela de quase 15 horas, a entrega depende de um replay manual, restrito à role ADMIN | TRANSCRICAO | [09:36] Larissa |
| ADR-003-NEG-02 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Consequência negativa | O aviso por email ao cliente ficou fora desta fase | TRANSCRICAO | [09:37] Larissa; [09:48] Larissa |
| ADR-003-TO | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Trade-off | Desistir da entrega automática depois de quase 15 horas, com replay manual a partir daí, em troca de cobrir indisponibilidades longas sem eventos presos | TRANSCRICAO | [09:15] Diego; [09:17] Diego; [09:18] Diego |
| ADR-004 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Decisão | Cada requisição é assinada com HMAC-SHA256 sobre o corpo, no header X-Signature; cada endpoint tem a sua secret, gerada pela plataforma, rotacionável, com a antiga válida por 24 horas | TRANSCRICAO | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia; [09:31] Marcos; [09:44] Diego |
| ADR-004-CTX-01 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Contexto | Os webhooks levam dados para fora da infraestrutura; o cliente precisa confirmar a origem e a integridade | TRANSCRICAO | [09:19] Sofia |
| ADR-004-CTX-02 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Contexto | Um cliente já expôs a secret dele em log de aplicação | TRANSCRICAO | [09:22] Diego |
| ADR-004-ALT-01 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Alternativa descartada | Secret global da plataforma: o vazamento dela comprometeria todos os endpoints | TRANSCRICAO | [09:21] Sofia |
| ADR-004-POS-01 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Consequência positiva | O cliente verifica a origem e a integridade de cada requisição do lado dele | TRANSCRICAO | [09:20] Sofia |
| ADR-004-POS-02 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Consequência positiva | HMAC-SHA256 é padrão de mercado, com biblioteca disponível para os clientes | TRANSCRICAO | [09:20] Sofia |
| ADR-004-POS-03 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Consequência positiva | O vazamento de uma secret afeta apenas um endpoint | TRANSCRICAO | [09:21] Sofia |
| ADR-004-POS-04 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Consequência positiva | Depois de uma rotação, o cliente tem 24 horas para migrar os sistemas dele | TRANSCRICAO | [09:21] Sofia |
| ADR-004-NEG-01 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Consequência negativa | A implementação de HMAC e da geração de secret passa por revisão de segurança antes do deploy, com pelo menos dois dias úteis reservados | TRANSCRICAO | [09:46] Sofia |
| ADR-004-NEG-02 | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Consequência negativa | A plataforma gera a secret de cada endpoint, guarda-a na configuração do webhook e oferece a rotação | TRANSCRICAO | [09:21] Bruno; [09:21] Sofia; [09:31] Marcos |
| ADR-004-TO | `docs/adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md` | Trade-off | Gerenciar uma secret por endpoint, com rotação e janela de 24 horas, em troca de limitar o dano de um vazamento a um endpoint | TRANSCRICAO | [09:21] Sofia |
| ADR-005 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Decisão | Entrega at-least-once; cada evento recebe um UUID ao entrar na outbox, enviado no header X-Event-Id, e o cliente deduplica por ele | TRANSCRICAO | [09:24] Diego; [09:25] Diego; [09:26] Larissa |
| ADR-005-CTX-01 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Contexto | O cliente pode receber o mesmo evento mais de uma vez, e era preciso decidir como ele distingue uma repetição | TRANSCRICAO | [09:24] Diego; [09:25] Bruno |
| ADR-005-ALT-01 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Alternativa descartada | Garantia exactly-once: exigiria coordenação entre a plataforma e o cliente, com muito mais complexidade | TRANSCRICAO | [09:25] Diego |
| ADR-005-POS-01 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência positiva | É o modelo que o mercado já usa, como Stripe e GitHub | TRANSCRICAO | [09:25] Diego |
| ADR-005-POS-02 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência positiva | Na avaliação de Diego, at-least-once com event_id resolve 99% dos casos | TRANSCRICAO | [09:25] Diego |
| ADR-005-POS-03 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência positiva | Não exige coordenação entre os dois lados | TRANSCRICAO | [09:25] Diego |
| ADR-005-NEG-01 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência negativa | A responsabilidade de deduplicar fica com o cliente | TRANSCRICAO | [09:25] Sofia |
| ADR-005-NEG-02 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Consequência negativa | Marcos se dispôs a documentar o comportamento com destaque no portal de desenvolvedor | TRANSCRICAO | [09:26] Marcos |
| ADR-005-TO | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Trade-off | O cliente recebe eventos repetidos e deduplica, em troca de evitar a complexidade do exactly-once | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| ADR-006 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | O módulo de webhooks reaproveita ao máximo o que já existe no projeto | TRANSCRICAO | [09:30] Larissa |
| ADR-006-CTX-01 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Contexto | O projeto tem um padrão claro de módulo, com classes de erro, logger Pino e middleware de erro centralizado | TRANSCRICAO | [09:27] Bruno; [09:28] Bruno; [09:29] Bruno |
| ADR-006-PAD-01 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | Estrutura de módulo: um módulo webhooks em src/modules/, com a mesma divisão de arquivos | TRANSCRICAO | [09:27] Bruno |
| ADR-006-PAD-02 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | Classes de erro: os erros do módulo derivam de AppError | TRANSCRICAO | [09:28] Bruno |
| ADR-006-PAD-03 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | Códigos de erro com o prefixo WEBHOOK_ em todos os códigos do módulo | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| ADR-006-PAD-04 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | Logger: o mesmo Pino, sem biblioteca nova | TRANSCRICAO | [09:29] Bruno |
| ADR-006-PAD-05 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | Tratamento de erro: o middleware de erro é usado como está | TRANSCRICAO | [09:29] Bruno |
| ADR-006-PAD-06 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | Validação: schemas Zod para as entradas do módulo | TRANSCRICAO | [09:30] Larissa |
| ADR-006-PAD-07 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | Controle de acesso: requireRole exige a role ADMIN no replay da DLQ | TRANSCRICAO | [09:36] Larissa |
| ADR-006-PAD-08 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Decisão | Identificadores: id da outbox em UUID, como o resto do projeto | TRANSCRICAO | [09:51] Larissa |
| ADR-006-ALT-01 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Alternativa descartada | Id auto incremental na outbox: descartado para seguir o padrão do projeto | TRANSCRICAO | [09:51] Diego; [09:51] Larissa |
| ADR-006-ALT-02 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Alternativa plausível, não discutida na reunião | Convenções próprias para o módulo; a reunião foi na direção oposta, de reuso máximo | TRANSCRICAO | [09:30] Larissa |
| ADR-006-POS-01 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Consequência positiva | Os erros do módulo são tratados pelo middleware de erro sem mudança nele | TRANSCRICAO | [09:29] Bruno |
| ADR-006-POS-02 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Consequência positiva | Nenhuma biblioteca nova de log entra no projeto | TRANSCRICAO | [09:29] Bruno |
| ADR-006-POS-03 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Consequência positiva | A regra de acesso do replay usa um middleware que já existe | TRANSCRICAO | [09:36] Larissa |
| ADR-006-NEG-01 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Consequência negativa | Quatro classes de erro têm código fixo, então as recusas de autenticação e de role saem sem o prefixo WEBHOOK_ | CODIGO | `src/shared/errors/http-errors.ts`; `src/middlewares/auth.middleware.ts` |
| ADR-006-NEG-02 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Consequência negativa | Toda falha de schema Zod sai como VALIDATION_ERROR; o código da recusa de URL http fica como questão em aberto | CODIGO | `src/middlewares/validate.middleware.ts` |
| ADR-006-NEG-03 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Consequência negativa | O logger tem nome de serviço fixo, e a lista de campos mascarados não cobre secret | CODIGO | `src/shared/logger/index.ts` |
| ADR-006-NEG-04 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Consequência negativa | A exigência de role ADMIN no replay pode ser contornada por autocadastro | CODIGO | `src/modules/auth/auth.routes.ts`; `src/modules/auth/auth.schemas.ts`; `src/modules/users/user.service.ts`; `src/modules/auth/auth.service.ts`; `src/middlewares/auth.middleware.ts` |
| ADR-006-NEG-05 | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Consequência negativa | "Tudo é uuid" vale para os models de domínio; OrderNumberSequence usa id inteiro | CODIGO | `prisma/schema.prisma` |
| ADR-006-TO | `docs/adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md` | Trade-off | As limitações dos componentes como estão hoje, em troca de um módulo igual aos outros, sem biblioteca nova de log | TRANSCRICAO | [09:29] Bruno; [09:30] Larissa |
| ADR-007 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Decisão | O payload é montado e gravado na outbox no momento da inserção, como snapshot do pedido naquela mudança de status | TRANSCRICAO | [09:52] Larissa; [09:52] Diego; [09:52] Bruno |
| ADR-007-CTX-01 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Contexto | A dúvida era se a outbox guarda o payload pronto ou só a referência ao pedido | TRANSCRICAO | [09:51] Bruno |
| ADR-007-CTX-02 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Contexto | O pedido pode mudar depois que o evento foi gravado | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-01 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Alternativa descartada | Guardar só o identificador do pedido e montar o payload no envio: o evento deixaria de refletir o estado de quando o status mudou | TRANSCRICAO | [09:52] Larissa |
| ADR-007-POS-01 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Consequência positiva | Cada evento descreve o pedido como ele estava na mudança de status, mesmo que o pedido mude depois | TRANSCRICAO | [09:52] Larissa |
| ADR-007-NEG-01 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Consequência negativa | A reunião não apontou custo; o que distingue a escolha da outra opção da pergunta de Bruno é que cada linha guarda o payload inteiro, e não só o identificador do pedido | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa |
| ADR-007-TO | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Trade-off | Gravar o payload completo em cada linha, em troca de o evento descrever o pedido como estava quando o status mudou | TRANSCRICAO | [09:51] Bruno; [09:52] Larissa |
| FDD-CTX-01 | `docs/FDD.md` | Contexto | A mudança de status acontece em OrderService.changeStatus, dentro de um único $transaction, e a aplicação não tem fila nem mecanismo de evento | CODIGO | `src/modules/orders/order.service.ts`; `package.json` |
| FDD-CTX-02 | `docs/FDD.md` | Contexto | Os clientes descobrem a mudança de status consultando a API | TRANSCRICAO | [09:00] Marcos |
| FDD-OBJ-01 | `docs/FDD.md` | Objetivo técnico | Gravar o evento na mesma transação da mudança de status | TRANSCRICAO | [09:06] Diego |
| FDD-OBJ-02 | `docs/FDD.md` | Objetivo técnico | Ler a outbox a cada 2 segundos, dentro do limite de 10 segundos aceito pelos clientes | TRANSCRICAO | [09:02] Marcos; [09:09] Diego |
| FDD-OBJ-03 | `docs/FDD.md` | Objetivo técnico | Entregar com garantia at-least-once, com um identificador por evento para o cliente deduplicar | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| FDD-OBJ-04 | `docs/FDD.md` | Objetivo técnico | Assinar cada requisição para o cliente validar origem e integridade | TRANSCRICAO | [09:19] Sofia; [09:20] Sofia |
| FDD-OBJ-05 | `docs/FDD.md` | Objetivo técnico | Retentar entregas com falha por quase 15 horas e, depois, mover o evento para uma DLQ com replay manual | TRANSCRICAO | [09:15] Diego; [09:17] Diego; [09:18] Diego |
| FDD-OBJ-06 | `docs/FDD.md` | Objetivo técnico | Seguir a estrutura de módulo, as classes de erro, o logger, o middleware de erro e os schemas Zod do projeto | TRANSCRICAO | [09:30] Larissa |
| FDD-EXC-01 | `docs/FDD.md` | Exclusão | Webhooks de entrada: o fluxo é só de saída | TRANSCRICAO | [09:02] Marcos |
| FDD-EXC-02 | `docs/FDD.md` | Exclusão | Garantia exactly-once: descartada | TRANSCRICAO | [09:25] Diego |
| FDD-EXC-03 | `docs/FDD.md` | Exclusão | Garantia de ordem global: sem garantia, limitação conhecida | TRANSCRICAO | [09:13] Larissa |
| FDD-EXC-04 | `docs/FDD.md` | Exclusão | Vários workers em paralelo: adiados | TRANSCRICAO | [09:13] Diego |
| FDD-EXC-05 | `docs/FDD.md` | Exclusão | Arquivamento das linhas entregues da outbox: fora do escopo desta feature | TRANSCRICAO | [09:08] Diego |
| FDD-EXC-06 | `docs/FDD.md` | Exclusão | Alerta por email ao cliente: próxima fase | TRANSCRICAO | [09:37] Larissa; [09:48] Larissa |
| FDD-EXC-07 | `docs/FDD.md` | Exclusão | Rate limiting de envio: observar e decidir depois | TRANSCRICAO | [09:39] Diego; [09:39] Larissa |
| FDD-EXC-08 | `docs/FDD.md` | Exclusão | Painel visual: só endpoints | TRANSCRICAO | [09:40] Larissa |
| FDD-EXC-09 | `docs/FDD.md` | Exclusão | Restrição de roles no CRUD de configuração: aberto a qualquer role autenticada nesta fase, e pode ser endurecido mais à frente | TRANSCRICAO | [09:37] Sofia |
| FDD-DADOS-01 | `docs/FDD.md` | Modelo de dados | As tabelas novas seguem o padrão do schema: id UUID, campos em camelCase e tabela em snake_case via @@map | CODIGO | `prisma/schema.prisma` |
| FDD-DADOS-02 | `docs/FDD.md` | Modelo de dados | Tabela de configuração dos webhooks, com url, secret, customer_id, estado ativo e lista de status | TRANSCRICAO | [09:21] Bruno; [09:21] Sofia; [09:31] Marcos |
| FDD-DADOS-03 | `docs/FDD.md` | Modelo de dados | Rotação: a secret antiga continua válida por 24 horas | TRANSCRICAO | [09:21] Sofia |
| FDD-DADOS-04 | `docs/FDD.md` | Modelo de dados | Tabela webhook_outbox, com o evento, o status, o payload gravado pronto e a data de criação | TRANSCRICAO | [09:06] Diego; [09:08] Diego; [09:52] Larissa |
| FDD-DADOS-05 | `docs/FDD.md` | Modelo de dados | Tabela webhook_dead_letter, separada, com payload, motivo da falha e timestamp | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-06 | `docs/FDD.md` | Modelo de dados | Histórico de entregas, com sucesso ou falha, payload, response e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-07 | `docs/FDD.md` | Modelo de dados | A outbox tem quatro status: pendente, processando, falhou e entregue | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-08 | `docs/FDD.md` | Modelo de dados | A outbox tem índice em status e em createdAt | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-09 | `docs/FDD.md` | Modelo de dados | Cada webhook guarda a lista dos status de pedido que quer receber | TRANSCRICAO | [09:33] Marcos |
| FDD-FLUXO-01 | `docs/FDD.md` | Fluxo | Bruno propôs a função publishWebhookEvent(tx, order, fromStatus, toStatus), que recebe o client da transação, sem injetar um repository inteiro no OrderService | TRANSCRICAO | [09:41] Bruno; [09:41] Diego |
| FDD-FLUXO-02 | `docs/FDD.md` | Fluxo | O changeStatus valida a transição, ajusta o estoque quando é o caso, atualiza o pedido e grava o histórico | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-FLUXO-03 | `docs/FDD.md` | Fluxo | Sem webhook do cliente inscrito no novo status, nada é inserido na outbox | TRANSCRICAO | [09:34] Bruno |
| FDD-FLUXO-04 | `docs/FDD.md` | Fluxo | O payload é montado com os dados do pedido naquele momento | TRANSCRICAO | [09:52] Larissa |
| FDD-FLUXO-05 | `docs/FDD.md` | Fluxo | Se a inserção na outbox falha, a transação inteira é desfeita, e o status não muda | TRANSCRICAO | [09:40] Bruno |
| FDD-FLUXO-06 | `docs/FDD.md` | Fluxo | O payload tem limite de 64 KB, com erro caso ultrapasse | TRANSCRICAO | [09:24] Larissa |
| FDD-FLUXO-07 | `docs/FDD.md` | Limitação | Nenhuma transição de pedido leva ao status PENDING, e a criação do pedido não passa pelo changeStatus: um webhook inscrito nesse status nunca dispara | CODIGO | `src/modules/orders/order.status.ts`; `src/modules/orders/order.service.ts` |
| FDD-FLUXO-08 | `docs/FDD.md` | Fluxo | O worker é um processo separado da API, único, com instância própria do PrismaClient e a mesma DATABASE_URL | TRANSCRICAO | [09:11] Diego; [09:12] Diego; [09:30] Bruno |
| FDD-FLUXO-09 | `docs/FDD.md` | Fluxo | A cada 2 segundos, o worker busca os eventos pendentes mais antigos, em lote pequeno | TRANSCRICAO | [09:08] Diego; [09:09] Diego |
| FDD-FLUXO-10 | `docs/FDD.md` | Fluxo | Os eventos são processados na ordem de createdAt | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-11 | `docs/FDD.md` | Fluxo | O envio tem timeout de 10 segundos | TRANSCRICAO | [09:42] Diego |
| FDD-FLUXO-12 | `docs/FDD.md` | Fluxo | Cada tentativa é registrada, com sucesso ou falha, payload, response e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| FDD-FLUXO-13 | `docs/FDD.md` | Fluxo | Retry: envio inicial mais 5 retentativas, com esperas de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| FDD-FLUXO-14 | `docs/FDD.md` | Fluxo | Se a 5ª retentativa falha, o evento é falha permanente e é movido para a DLQ | TRANSCRICAO | [09:15] Diego |
| FDD-FLUXO-15 | `docs/FDD.md` | Fluxo | O worker grava na DLQ o payload, o motivo da falha e o timestamp | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-16 | `docs/FDD.md` | Fluxo | O reprocessamento é manual: o endpoint de replay recoloca o evento na outbox como pendente | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-17 | `docs/FDD.md` | Fluxo | O replay exige a role ADMIN e registra em log quem o executou, para auditoria | TRANSCRICAO | [09:36] Sofia; [09:36] Larissa |
| FDD-CONV-01 | `docs/FDD.md` | Convenção | Todas as rotas ficam sob /api/v1 | CODIGO | `src/app.ts` |
| FDD-CONV-02 | `docs/FDD.md` | Convenção | Todas as rotas exigem autenticação por JWT | TRANSCRICAO | [09:32] Larissa |
| FDD-CONV-03 | `docs/FDD.md` | Convenção | Corpo e resposta em JSON, com campos em camelCase | CODIGO | `src/modules/orders/order.schemas.ts` |
| FDD-CONV-04 | `docs/FDD.md` | Convenção | Erros no formato { error: { code, message, details } } | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-CONV-05 | `docs/FDD.md` | Convenção | Listas paginadas com page e pageSize, e resposta com data e pagination | CODIGO | `src/modules/orders/order.schemas.ts`; `src/shared/http/response.ts` |
| FDD-CONV-06 | `docs/FDD.md` | Convenção | Criação responde 201, leitura e edição 200, remoção 204 | CODIGO | `src/modules/orders/order.controller.ts` |
| FDD-CONTRATO-01 | `docs/FDD.md` | Contrato | POST /api/v1/webhooks cadastra um webhook; a secret é gerada e devolvida na criação, e a URL precisa ser https | TRANSCRICAO | [09:23] Sofia; [09:31] Marcos; [09:37] Sofia |
| FDD-CONTRATO-02 | `docs/FDD.md` | Contrato | GET /api/v1/webhooks?customerId= lista os webhooks de um cliente | TRANSCRICAO | [09:33] Bruno; [09:37] Sofia |
| FDD-CONTRATO-03 | `docs/FDD.md` | Contrato | PATCH /api/v1/webhooks/:id edita um webhook | TRANSCRICAO | [09:33] Bruno; [09:37] Sofia |
| FDD-CONTRATO-04 | `docs/FDD.md` | Contrato | DELETE /api/v1/webhooks/:id remove um webhook | TRANSCRICAO | [09:33] Bruno; [09:37] Sofia |
| FDD-CONTRATO-05 | `docs/FDD.md` | Contrato | POST /api/v1/webhooks/:id/secret/rotate gera uma secret nova; a antiga continua válida por 24 horas | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | `docs/FDD.md` | Contrato | GET /api/v1/webhooks/:id/deliveries traz o histórico de entregas, com sucesso ou falha, payload, response e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-07 | `docs/FDD.md` | Contrato | POST /api/v1/admin/webhooks/dead-letter/:id/replay recoloca um evento da DLQ na outbox, exige a role ADMIN e registra quem executou | TRANSCRICAO | [09:35] Diego; [09:36] Sofia; [09:36] Larissa |
| FDD-CONTRATO-08 | `docs/FDD.md` | Contrato | O identificador do cliente não vem do JWT e vai no body ou no path, sem escolha na reunião | TRANSCRICAO | [09:32] Larissa |
| FDD-HEADER-01 | `docs/FDD.md` | Contrato | Header X-Event-Id: UUID do evento, gerado quando ele entra na outbox | TRANSCRICAO | [09:25] Diego; [09:44] Diego |
| FDD-HEADER-02 | `docs/FDD.md` | Contrato | Header X-Signature: HMAC-SHA256 sobre o corpo do request, com a secret do endpoint | TRANSCRICAO | [09:20] Sofia; [09:22] Sofia; [09:44] Diego |
| FDD-HEADER-03 | `docs/FDD.md` | Contrato | Header X-Timestamp: timestamp do envio, para o cliente detectar replay attack | TRANSCRICAO | [09:44] Diego |
| FDD-HEADER-04 | `docs/FDD.md` | Contrato | Header X-Webhook-Id: id do webhook cadastrado | TRANSCRICAO | [09:44] Sofia; [09:45] Diego |
| FDD-HEADER-05 | `docs/FDD.md` | Contrato | Header Content-Type: application/json | TRANSCRICAO | [09:44] Diego |
| FDD-PAYLOAD-01 | `docs/FDD.md` | Contrato | Payload em snake_case: event_id, event_type, timestamp, order_id, order_number, from_status, to_status, customer_id e campos básicos do pedido, como total_cents | TRANSCRICAO | [09:43] Diego |
| FDD-PAYLOAD-02 | `docs/FDD.md` | Contrato | Os itens do pedido não vão no payload; quem quiser detalhes consulta o GET /orders/:id | TRANSCRICAO | [09:43] Diego |
| FDD-PAYLOAD-03 | `docs/FDD.md` | Contrato | O payload tem limite de 64 KB | TRANSCRICAO | [09:24] Larissa |
| FDD-PAYLOAD-04 | `docs/FDD.md` | Contrato | O cliente pode receber o mesmo evento mais de uma vez e deduplica pelo X-Event-Id | TRANSCRICAO | [09:24] Diego; [09:25] Diego |
| FDD-ERRO-01 | `docs/FDD.md` | Erro | Os erros do módulo derivam de AppError e usam o prefixo WEBHOOK_ | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa |
| FDD-ERRO-02 | `docs/FDD.md` | Erro | WEBHOOK_NOT_FOUND: o webhook do :id não existe; o status 404 é o de NotFoundError no código | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-03 | `docs/FDD.md` | Erro | WEBHOOK_INVALID_URL: a URL não é https; o status 400 é o de BadRequestError no código | TRANSCRICAO | [09:23] Sofia; [09:28] Bruno |
| FDD-ERRO-04 | `docs/FDD.md` | Erro | Erro quando o payload passa de 64 KB; o código WEBHOOK_PAYLOAD_TOO_LARGE e o status 422 são da Proposta P-16 | TRANSCRICAO | [09:24] Larissa |
| FDD-ERRO-05 | `docs/FDD.md` | Erro | WEBHOOK_SECRET_REQUIRED, citado como exemplo, fica fora da matriz pela Proposta P-16: depois a reunião definiu que a secret é gerada pela plataforma | TRANSCRICAO | [09:28] Bruno; [09:31] Marcos |
| FDD-ERRO-06 | `docs/FDD.md` | Erro | VALIDATION_ERROR, 400: falha de schema Zod no body, na query ou nos params | CODIGO | `src/middlewares/validate.middleware.ts` |
| FDD-ERRO-07 | `docs/FDD.md` | Erro | UNAUTHORIZED, 401: token ausente, inválido ou expirado | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-ERRO-08 | `docs/FDD.md` | Erro | FORBIDDEN, 403: role diferente de ADMIN no replay | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-ERRO-09 | `docs/FDD.md` | Erro | INTERNAL_SERVER_ERROR, 500: erro não tratado | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-ERRO-10 | `docs/FDD.md` | Erro | O middleware de erro já trata AppError, Zod e Prisma, e não precisa mudar | TRANSCRICAO | [09:29] Bruno |
| FDD-RES-01 | `docs/FDD.md` | Resiliência | Atomicidade: o evento é gravado na transação do changeStatus; se a inserção falha, o status não muda | TRANSCRICAO | [09:06] Diego; [09:40] Bruno |
| FDD-RES-02 | `docs/FDD.md` | Resiliência | Isolamento: o envio HTTP fica fora da transação, em outro processo | TRANSCRICAO | [09:06] Diego; [09:11] Diego |
| FDD-RES-03 | `docs/FDD.md` | Resiliência | Timeout: 10 segundos por chamada; sem resposta, a entrega conta como falha | TRANSCRICAO | [09:42] Diego |
| FDD-RES-04 | `docs/FDD.md` | Resiliência | Retry com backoff: 5 retentativas | TRANSCRICAO | [09:17] Diego; [09:17] Larissa |
| FDD-RES-05 | `docs/FDD.md` | Resiliência | DLQ: esgotadas as retentativas, o evento vai para webhook_dead_letter, com payload e motivo da falha | TRANSCRICAO | [09:15] Diego; [09:18] Diego |
| FDD-RES-06 | `docs/FDD.md` | Resiliência | Fallback: não há fallback automático nesta fase; o replay é manual, e o email ficou para a próxima fase | TRANSCRICAO | [09:18] Diego; [09:48] Larissa |
| FDD-RES-07 | `docs/FDD.md` | Resiliência | Deduplicação: o cliente descarta repetições pelo X-Event-Id | TRANSCRICAO | [09:25] Diego |
| FDD-OBS-01 | `docs/FDD.md` | Observabilidade | O projeto usa Pino, e nada novo entra | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | `docs/FDD.md` | Observabilidade | O replay registra quem o executou | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-03 | `docs/FDD.md` | Observabilidade | O projeto não tem biblioteca de métricas nem de tracing | CODIGO | `package.json` |
| FDD-OBS-04 | `docs/FDD.md` | Observabilidade | Os logs usam o logger do projeto, com nome de evento em snake_case, como os que o projeto já emite | CODIGO | `src/shared/logger/index.ts`; `src/server.ts`; `src/middlewares/request-logger.middleware.ts` |
| FDD-OBS-05 | `docs/FDD.md` | Observabilidade | A reunião registrou um caso de secret vazada em log de aplicação do cliente | TRANSCRICAO | [09:22] Diego |
| FDD-OBS-06 | `docs/FDD.md` | Observabilidade | Na API, cada requisição já recebe um X-Request-Id, gravado no log http_request | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-INT-01 | `docs/FDD.md` | Integração | order.service.ts: o changeStatus chama publishWebhookEvent dentro do $transaction | TRANSCRICAO | [09:40] Bruno; [09:41] Bruno |
| FDD-INT-02 | `docs/FDD.md` | Integração | schema.prisma: ganha os models das tabelas novas, no padrão de id UUID | TRANSCRICAO | [09:51] Larissa |
| FDD-INT-03 | `docs/FDD.md` | Integração | app.ts: o buildControllers instancia o repository, o service e o controller do módulo | CODIGO | `src/app.ts` |
| FDD-INT-04 | `docs/FDD.md` | Integração | routes/index.ts: o tipo Controllers ganha o controller do módulo, e o buildApiRouter monta os routers novos | CODIGO | `src/routes/index.ts` |
| FDD-INT-05 | `docs/FDD.md` | Integração | server.ts: serve de modelo para o entry point do worker | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-06 | `docs/FDD.md` | Integração | config/database.ts: o worker importa o prisma exportado pelo arquivo; por rodar em outro processo, é outra instância | TRANSCRICAO | [09:30] Bruno |
| FDD-INT-07 | `docs/FDD.md` | Integração | config/env.ts: o arquivo exige JWT_SECRET e encerra o processo sem ela, então o ambiente do worker precisa da variável | CODIGO | `src/config/env.ts` |
| FDD-INT-08 | `docs/FDD.md` | Integração | Classes de erro: as do módulo derivam de AppError | TRANSCRICAO | [09:28] Bruno |
| FDD-INT-09 | `docs/FDD.md` | Integração | auth.middleware.ts: authenticate nas rotas do módulo e requireRole de ADMIN no replay | TRANSCRICAO | [09:36] Larissa |
| FDD-INT-10 | `docs/FDD.md` | Integração | validate.middleware.ts: valida as entradas com os schemas Zod do módulo | TRANSCRICAO | [09:30] Larissa |
| FDD-INT-11 | `docs/FDD.md` | Integração | error.middleware.ts: sem mudança | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-12 | `docs/FDD.md` | Integração | logger: o mesmo logger no módulo e no worker | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-13 | `docs/FDD.md` | Integração | response.ts: paginated() monta a resposta das listagens | CODIGO | `src/shared/http/response.ts` |
| FDD-INT-14 | `docs/FDD.md` | Integração | package.json: ganha o script worker | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-15 | `docs/FDD.md` | Integração | tests/setup.ts: o beforeEach limpa as tabelas uma a uma, e as tabelas novas entram nessa limpeza | CODIGO | `tests/setup.ts` |
| FDD-INT-16 | `docs/FDD.md` | Integração | Arquivo novo worker.ts, o entry point do worker, em src/ | TRANSCRICAO | [09:11] Larissa; [09:28] Bruno |
| FDD-INT-17 | `docs/FDD.md` | Integração | Pasta nova webhooks em src/modules/, com controller, service, repository, routes e schemas | TRANSCRICAO | [09:27] Bruno |
| FDD-INT-18 | `docs/FDD.md` | Integração | Arquivo de processamento na pasta do módulo; a reunião deixou o nome entre webhook.worker.ts e webhook.processor.ts | TRANSCRICAO | [09:28] Bruno |
| FDD-DEP-01 | `docs/FDD.md` | Dependência | Banco: o MySQL que já existe, sem infraestrutura nova | TRANSCRICAO | [09:07] Diego |
| FDD-DEP-02 | `docs/FDD.md` | Dependência | Migrations feitas com o Prisma, pelo script db:migrate | CODIGO | `package.json` |
| FDD-DEP-03 | `docs/FDD.md` | Dependência | Runtime: Node 20 ou superior | CODIGO | `package.json` |
| FDD-DEP-04 | `docs/FDD.md` | Dependência | Logger: Pino, já presente | TRANSCRICAO | [09:29] Bruno |
| FDD-DEP-05 | `docs/FDD.md` | Dependência | Processos: passam a ser dois, a API e o worker, cada um com o seu script | TRANSCRICAO | [09:11] Diego; [09:11] Larissa |
| FDD-DEP-06 | `docs/FDD.md` | Dependência | Os testes existentes rodam contra o MySQL real e dependem da limpeza de tests/setup.ts | CODIGO | `tests/setup.ts` |
| FDD-DEP-07 | `docs/FDD.md` | Dependência | O código de HMAC e de geração de secret passa por revisão de segurança antes do deploy | TRANSCRICAO | [09:46] Sofia |
| FDD-CA-01 | `docs/FDD.md` | Critério de aceite técnico | Uma mudança de status com webhook do cliente inscrito grava a linha na outbox na mesma transação | TRANSCRICAO | [09:06] Diego |
| FDD-CA-02 | `docs/FDD.md` | Critério de aceite técnico | Se a inserção na outbox falha, o status do pedido não muda | TRANSCRICAO | [09:40] Bruno |
| FDD-CA-03 | `docs/FDD.md` | Critério de aceite técnico | Sem webhook do cliente inscrito naquele status, nenhuma linha é inserida | TRANSCRICAO | [09:34] Bruno |
| FDD-CA-04 | `docs/FDD.md` | Critério de aceite técnico | O payload gravado reflete o pedido no momento da mudança, mesmo que o pedido mude depois | TRANSCRICAO | [09:52] Larissa |
| FDD-CA-05 | `docs/FDD.md` | Critério de aceite técnico | O worker roda em processo separado da API e lê a outbox a cada 2 segundos | TRANSCRICAO | [09:09] Diego; [09:11] Diego |
| FDD-CA-06 | `docs/FDD.md` | Critério de aceite técnico | A requisição enviada leva X-Event-Id, X-Signature, X-Timestamp, X-Webhook-Id e Content-Type application/json | TRANSCRICAO | [09:44] Diego; [09:44] Sofia |
| FDD-CA-07 | `docs/FDD.md` | Critério de aceite técnico | A assinatura confere com o HMAC-SHA256 do corpo, calculado com a secret do endpoint | TRANSCRICAO | [09:22] Sofia |
| FDD-CA-08 | `docs/FDD.md` | Critério de aceite técnico | Uma chamada sem resposta em 10 segundos conta como falha e entra em retry | TRANSCRICAO | [09:42] Diego |
| FDD-CA-09 | `docs/FDD.md` | Critério de aceite técnico | As retentativas seguem as esperas de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas | TRANSCRICAO | [09:17] Diego |
| FDD-CA-10 | `docs/FDD.md` | Critério de aceite técnico | Depois da 5ª retentativa com falha, o evento está em webhook_dead_letter, com payload e motivo | TRANSCRICAO | [09:15] Diego; [09:18] Diego |
| FDD-CA-11 | `docs/FDD.md` | Critério de aceite técnico | O replay sem role ADMIN responde 403; com ADMIN, recoloca o evento como pendente e registra quem executou | TRANSCRICAO | [09:18] Diego; [09:36] Sofia; [09:36] Larissa |
| FDD-CA-12 | `docs/FDD.md` | Critério de aceite técnico | O cadastro com URL http é recusado | TRANSCRICAO | [09:23] Sofia |
| FDD-CA-13 | `docs/FDD.md` | Critério de aceite técnico | A secret é devolvida na criação e pode ser rotacionada pela API; a antiga vale por 24 horas | TRANSCRICAO | [09:31] Marcos; [09:21] Sofia |
| FDD-CA-14 | `docs/FDD.md` | Critério de aceite técnico | Um payload acima de 64 KB gera erro | TRANSCRICAO | [09:24] Larissa |
| FDD-CA-15 | `docs/FDD.md` | Critério de aceite técnico | Os erros do módulo usam códigos com o prefixo WEBHOOK_ | TRANSCRICAO | [09:29] Larissa |
| FDD-CA-16 | `docs/FDD.md` | Critério de aceite técnico | O histórico de entregas traz sucesso ou falha, payload, response e tempo de resposta | TRANSCRICAO | [09:34] Marcos |
| FDD-RISCO-01 | `docs/FDD.md` | Risco | A transação do changeStatus, que já é pesada, ganha uma leitura dos webhooks do cliente, pelo filtro na inserção, e uma escrita por webhook inscrito, pela Proposta P-02 | TRANSCRICAO | [09:04] Bruno; [09:34] Bruno; [09:40] Bruno |
| FDD-RISCO-02 | `docs/FDD.md` | Risco | O acúmulo de linhas na outbox pode deixar a leitura do worker lenta, dúvida levantada por Bruno; índices e lote pequeno | TRANSCRICAO | [09:07] Bruno; [09:08] Diego |
| FDD-RISCO-03 | `docs/FDD.md` | Risco | A secret aparece em log da plataforma: a lista de campos mascarados do logger não a cobre | CODIGO | `src/shared/logger/index.ts` |
| FDD-RISCO-05 | `docs/FDD.md` | Risco | A limpeza dos testes existentes falha por chave estrangeira se as tabelas novas não forem limpas antes | CODIGO | `tests/setup.ts` |
| FDD-RISCO-06 | `docs/FDD.md` | Risco | Respostas de autenticação e de role saem sem o prefixo WEBHOOK_ | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-P-01 | `docs/FDD.md` | Proposta | Nomes das tabelas webhooks e webhook_deliveries, dos campos e dos status em inglês; statuses em coluna Json; índice em customerId na tabela webhooks | CODIGO | `prisma/schema.prisma` |
| FDD-P-02 | `docs/FDD.md` | Proposta | Uma linha na outbox por webhook inscrito; o id da linha é o identificador do evento | TRANSCRICAO | [09:25] Diego; [09:44] Sofia |
| FDD-P-03 | `docs/FDD.md` | Proposta | Campos attempts e nextAttemptAt para agendar as retentativas dentro do polling; attempt 1 é o envio inicial | TRANSCRICAO | [09:09] Diego; [09:17] Diego |
| FDD-P-04 | `docs/FDD.md` | Proposta | O evento vira PROCESSING antes do envio; FAILED é falha com retentativa agendada; esgotadas as retentativas, a linha sai da outbox e vai para a DLQ | TRANSCRICAO | [09:08] Diego; [09:15] Diego; [09:18] Diego |
| FDD-P-05 | `docs/FDD.md` | Proposta | Tabela webhook_deliveries, com uma linha por tentativa; histórico paginado, e "os últimos 100 webhooks" lido como exemplo | TRANSCRICAO | [09:34] Marcos |
| FDD-P-06 | `docs/FDD.md` | Proposta | Campos previousSecret e previousSecretExpiresAt para a janela de 24 horas | TRANSCRICAO | [09:21] Sofia |
| FDD-P-07 | `docs/FDD.md` | Proposta | customerId no body do cadastro e, obrigatório, na query da listagem | TRANSCRICAO | [09:32] Larissa |
| FDD-P-08 | `docs/FDD.md` | Proposta | Rotação pela rota POST /api/v1/webhooks/:id/secret/rotate, com resposta 200 que traz a secret nova e a validade da antiga | TRANSCRICAO | [09:21] Sofia |
| FDD-P-09 | `docs/FDD.md` | Proposta | A secret só aparece nas respostas de criação e de rotação | TRANSCRICAO | [09:31] Marcos |
| FDD-P-10 | `docs/FDD.md` | Proposta | Rotação e histórico de entregas com a mesma regra de acesso do CRUD | TRANSCRICAO | [09:37] Sofia |
| FDD-P-11 | `docs/FDD.md` | Proposta | O webhook nasce ativo; inativo, não gera eventos novos; url, statuses e active são editáveis | TRANSCRICAO | [09:21] Bruno; [09:21] Sofia; [09:33] Bruno |
| FDD-P-12 | `docs/FDD.md` | Proposta | Requisição ao cliente por POST; X-Signature em hexadecimal; X-Timestamp em ISO 8601 | TRANSCRICAO | [09:44] Diego |
| FDD-P-13 | `docs/FDD.md` | Proposta | Sucesso é resposta 2xx dentro do timeout; qualquer outro resultado é falha | TRANSCRICAO | [09:42] Diego |
| FDD-P-14 | `docs/FDD.md` | Proposta | O replay recoloca o evento a partir do payload guardado na DLQ, com um ciclo novo de tentativas, e responde 200 | TRANSCRICAO | [09:18] Diego |
| FDD-P-15 | `docs/FDD.md` | Proposta | Ao iniciar, o worker devolve a PENDING os eventos em PROCESSING; ao receber SIGINT ou SIGTERM, encerra como o servidor | TRANSCRICAO | [09:11] Diego; [09:12] Diego |
| FDD-P-16 | `docs/FDD.md` | Proposta | Códigos WEBHOOK_DEAD_LETTER_NOT_FOUND (404) e WEBHOOK_PAYLOAD_TOO_LARGE (422); regra de https verificada no service; classes de erro novas no arquivo das existentes; WEBHOOK_SECRET_REQUIRED fora da matriz | TRANSCRICAO | [09:28] Bruno; [09:29] Larissa; [09:31] Marcos |
| FDD-P-17 | `docs/FDD.md` | Proposta | Arquivo de processamento com o nome webhook.processor.ts | TRANSCRICAO | [09:28] Bruno |
| FDD-P-18 | `docs/FDD.md` | Proposta | Eventos de log do módulo e do worker; *.secret e *.previousSecret na lista de campos mascarados | CODIGO | `src/shared/logger/index.ts` |
| FDD-P-19 | `docs/FDD.md` | Proposta | Métricas tiradas das tabelas e dos logs, sem biblioteca nova | CODIGO | `package.json` |
| FDD-P-20 | `docs/FDD.md` | Proposta | Correlação por identificadores no lugar de tracing distribuído | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-P-21 | `docs/FDD.md` | Proposta | fetch e crypto do Node, sem dependência nova | CODIGO | `package.json` |
| FDD-P-22 | `docs/FDD.md` | Proposta | Limite de 64 KB verificado na inserção; o erro desfaz a mudança de status | TRANSCRICAO | [09:24] Larissa; [09:40] Bruno |
| FDD-P-23 | `docs/FDD.md` | Proposta | O worker processa um evento de cada vez | TRANSCRICAO | [09:12] Diego |
| FDD-P-24 | `docs/FDD.md` | Proposta | Cadastro com cliente inexistente responde 404 WEBHOOK_CUSTOMER_NOT_FOUND | TRANSCRICAO | [09:29] Larissa |
| FDD-P-25 | `docs/FDD.md` | Proposta | Nenhuma rota existente muda de contrato, salvo o PATCH /api/v1/orders/:id/status, que ganha o erro de payload acima do limite | CODIGO | `src/modules/orders/order.routes.ts` |
| FDD-P-26 | `docs/FDD.md` | Proposta | Chave estrangeira em webhooks.customerId e nas colunas webhookId; orderId sem chave estrangeira na outbox e na DLQ, e eventId sem chave estrangeira nas tentativas | CODIGO | `prisma/schema.prisma`; `src/modules/orders/order.repository.ts` |
| FDD-P-27 | `docs/FDD.md` | Proposta | publishWebhookEvent fica no módulo de webhooks e é chamada depois da gravação do histórico | TRANSCRICAO | [09:41] Bruno |
| FDD-QA-01 | `docs/FDD.md` | Questão em aberto | Código de erro da URL http: a reunião quer a recusa no schema Zod e citou WEBHOOK_INVALID_URL | TRANSCRICAO | [09:23] Sofia; [09:28] Bruno |
| FDD-QA-02 | `docs/FDD.md` | Questão em aberto | Identificação do cliente: body ou path, sem escolha na reunião | TRANSCRICAO | [09:32] Larissa |
| FDD-QA-03 | `docs/FDD.md` | Questão em aberto | Tamanho do lote do worker: a reunião disse "batch pequeno", sem número | TRANSCRICAO | [09:08] Diego |
| FDD-QA-04 | `docs/FDD.md` | Questão em aberto | Campos básicos do pedido no payload: Diego citou total_cents como exemplo e não listou os demais | TRANSCRICAO | [09:43] Diego |
| FDD-QA-05 | `docs/FDD.md` | Questão em aberto | Eventos de um webhook removido ou desativado: a reunião não tratou dos eventos já gravados | TRANSCRICAO | [09:21] Bruno; [09:21] Sofia; [09:33] Bruno |
| FDD-QA-06 | `docs/FDD.md` | Questão em aberto | Remoção de cliente com webhooks cadastrados: CustomerService.delete apaga a linha do cliente | CODIGO | `src/modules/customers/customer.service.ts` |
| FDD-QA-07 | `docs/FDD.md` | Questão em aberto | Duas rotações de secret em menos de 24 horas: a reunião só definiu a janela de 24 horas para a secret antiga | TRANSCRICAO | [09:21] Sofia |
| FDD-QA-08 | `docs/FDD.md` | Questão em aberto | Registro da DLQ depois do replay: a reunião não disse se ele permanece, é marcado ou é removido | TRANSCRICAO | [09:18] Diego |

## Cobertura

| Documento | Itens identificáveis | Linhas |
| --- | --- | --- |
| `docs/PRD.md` | 89 | 88 |
| `docs/RFC.md` | 42 | 42 |
| `docs/adrs/` (7 ADRs) | 90 | 90 |
| `docs/FDD.md` | 171 | 170 |
| **Total** | **392** | **390** |

- **Cobertura:** 390 de 392 itens têm linha (99,5%).
- **Fonte `TRANSCRICAO`:** 340 de 390 linhas (87,2%), todas com minuto e falante que existem na transcrição.
- **Fonte `CODIGO`:** 50 de 390 linhas (12,8%), todas com caminho de arquivo que existe no repositório.

Dois itens ficam sem linha, e por isso os IDs `PRD-TESTE-03` e `FDD-RISCO-04` não existem:

- **PRD, seção 12, "Critérios técnicos".** É proposta do PRD, que remete aos critérios do FDD, e não tem fala nem arquivo como origem. Os 16 critérios estão nas linhas `FDD-CA-01` a `FDD-CA-16`.
- **FDD, seção 13, eventos presos em `PROCESSING`.** O risco nasce de uma proposta do próprio FDD, e não de uma fala ou de um arquivo. As propostas envolvidas estão nas linhas `FDD-P-04` e `FDD-P-15`.

Como os itens foram contados:

- Conta como item cada entrada de tabela ou de lista dos documentos que registra um requisito, objetivo, item fora de escopo, decisão, alternativa, consequência, trade-off, risco, questão em aberto, critério, contrato, erro, tabela do modelo, passo de fluxo, ponto de integração ou dependência, além das afirmações de contexto com origem citada.
- Não contam os metadados, o resumo executivo do RFC, as listas de links entre documentos, os exemplos de request e de response e as listas que só repetem itens de outra seção.
- Um ponto do FDD cuja única origem é uma proposta conta uma vez, na linha da proposta.
