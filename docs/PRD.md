# PRD: Sistema de Webhooks de Notificação de Pedidos

Este documento diz por que a feature existe e o que ela entrega. A proposta técnica está no [RFC](RFC.md), as decisões nos [ADRs](adrs/README.md) e o detalhamento de implementação no [FDD](FDD.md).

As afirmações trazem a sua origem: o minuto e o falante da reunião registrada em [TRANSCRICAO.md](../TRANSCRICAO.md), ou o caminho de um arquivo do repositório. A seção 8 aponta para os ADRs, onde está a origem de cada decisão. O que é avaliação ou proposta deste documento vem declarado como tal.

## 1. Resumo e contexto da feature

A plataforma é um Order Management System em que o pedido percorre um ciclo de status, de `PENDING` a `DELIVERED` ou `CANCELLED` (`prisma/schema.prisma`; `src/modules/orders/order.status.ts`). Hoje ela não tem nenhum mecanismo de notificação: nenhuma dependência de `package.json` cumpre esse papel.

Três clientes B2B, Atlas Comercial, MaxDistribuição e Nova Cargo, pediram formalmente para ser notificados em tempo real dessas mudanças ([09:00] Marcos). A feature entrega isso por webhooks de saída: a plataforma chama um endpoint do cliente a cada mudança de status que ele escolheu acompanhar ([09:02] Marcos; [09:33] Marcos).

## 2. Problema e motivação

- **Como os clientes fazem hoje.** Eles consultam o `GET /orders` de tempos em tempos para descobrir se algo mudou, o que deixa a integração “lenta e cara pra eles” ([09:00] Marcos).
- **O que eles pedem.** Notificação em tempo real. Para eles, “qualquer coisa abaixo de 10 segundos já é "tempo real"”; o importante é não ficarem atualizando manualmente ([09:02] Marcos).
- **O que está em jogo.** A Atlas sinalizou que pode migrar para um concorrente se a feature não for entregue ([09:00] Marcos) e pediu a entrega para o fim de novembro ([09:45] Marcos).

## 3. Público-alvo e cenários de uso

| Público | Quem é | Origem |
| --- | --- | --- |
| Clientes B2B integradores | Atlas Comercial, MaxDistribuição e Nova Cargo, que recebem as notificações nos seus sistemas | [09:00] Marcos |
| Usuários autenticados da API | Quem cadastra e administra os webhooks pela API, com o JWT do sistema | [09:32] Marcos; [09:32] Larissa |
| Administradores | Usuários com a role ADMIN, que reprocessam eventos da DLQ | [09:36] Sofia |

Cenários de uso:

1. **Deixar de consultar.** O cliente cadastra um webhook e passa a ser avisado das mudanças, em vez de consultar a API de tempos em tempos ([09:00] Marcos; [09:31] Marcos).
2. **Receber só o que interessa.** O cliente escolhe os status de cada webhook: “só quero saber quando vira SHIPPED e DELIVERED” ([09:33] Marcos).
3. **Ter mais de um webhook.** Um cliente com vários cadastros identifica em cada envio qual deles foi usado ([09:44] Sofia).
4. **Ficar fora do ar.** Um cliente em manutenção planejada, como a de duas horas que já aconteceu ([09:16] Diego), tem as entregas retentadas.
5. **Conferir o que foi enviado.** O cliente consulta o histórico de entregas de um webhook ([09:34] Marcos).
6. **Trocar a secret.** O cliente pede uma secret nova pela API e tem 24 horas para migrar os sistemas dele ([09:21] Sofia).
7. **Reprocessar.** Um administrador recoloca na outbox um evento que esgotou as retentativas ([09:18] Diego; [09:36] Sofia).

## 4. Objetivos e métricas de sucesso

| # | Objetivo | Métrica | Meta | Origem |
| --- | --- | --- | --- | --- |
| OBJ-01 | Notificar em tempo real | Tempo entre a mudança de status e o envio da notificação | Abaixo de 10 segundos | [09:02] Marcos |

Marcos não disse de onde até onde os 10 segundos são medidos. Medir da mudança de status até o envio da notificação é leitura deste documento.

A reunião não definiu outras metas, como taxa de entrega ou número de clientes usando a feature. O prazo pedido pela Atlas está na seção 9, e a janela de retentativas no requisito RNF-08.

## 5. Escopo

### Incluso

- Notificação por webhook a cada mudança de status de pedido, para os webhooks do cliente inscritos naquele status (RF-07 a RF-10; RF-16).
- Cadastro, edição, remoção e listagem de webhooks pela API (RF-01 a RF-05).
- Escolha, por webhook, dos status a receber (RF-06).
- Assinatura de cada requisição e rotação da secret (RNF-04; RF-11).
- Retentativas automáticas, DLQ e replay manual (RF-13 a RF-15).
- Histórico de entregas por webhook (RF-12).

### Fora de escopo

| Item | Situação | Origem |
| --- | --- | --- |
| Alerta por email quando o webhook do cliente falha | Adiado para a próxima fase | [09:37] Larissa; [09:48] Larissa |
| Painel visual para o cliente | Fora: só endpoints; painel é projeto de outro time | [09:40] Larissa |
| Rate limiting de envio | Observar e decidir depois | [09:39] Diego; [09:39] Larissa |
| Webhooks de entrada, do cliente para a plataforma | Fora: os clientes querem receber, não enviar | [09:02] Marcos |
| Garantia exactly-once | Descartada | [09:25] Diego |
| Garantia de ordem global | Sem garantia; os clientes não pediram | [09:13] Larissa; [09:14] Marcos |
| Vários workers em paralelo | Adiado | [09:13] Diego |
| Arquivamento das linhas entregues da outbox | Fora do escopo desta feature | [09:08] Diego |
| Restrição de roles no CRUD de configuração | Aberto a qualquer role autenticada nesta fase: “Mais pra frente a gente pode endurecer.” | [09:37] Sofia |

## 6. Requisitos funcionais

| # | Requisito | Origem |
| --- | --- | --- |
| RF-01 | Cadastrar um webhook pela API, informando a URL e a lista de status a receber. | [09:31] Marcos |
| RF-02 | Gerar a secret do webhook na plataforma e devolvê-la na criação. | [09:31] Marcos |
| RF-03 | Editar um webhook. | [09:33] Bruno |
| RF-04 | Remover um webhook. | [09:33] Bruno |
| RF-05 | Listar os webhooks de um cliente. | [09:33] Bruno |
| RF-06 | Escolher, por webhook, quais status geram notificação. | [09:33] Bruno; [09:33] Marcos |
| RF-07 | Notificar o cliente quando o status de um pedido dele muda para um status em que há webhook inscrito. | [09:00] Marcos; [09:34] Bruno |
| RF-08 | Enviar na notificação os dados do evento e do pedido, sem os itens do pedido. Os campos estão no [FDD](FDD.md). | [09:43] Diego |
| RF-09 | Identificar cada evento com um identificador único, para o cliente reconhecer repetições. | [09:25] Diego |
| RF-10 | Informar em cada envio qual webhook cadastrado o originou. | [09:44] Sofia; [09:45] Diego |
| RF-11 | Rotacionar a secret pela API, mantendo a antiga válida por 24 horas. | [09:21] Sofia |
| RF-12 | Consultar o histórico de entregas de um webhook, com sucesso ou falha, payload, response e tempo de resposta. | [09:34] Marcos |
| RF-13 | Retentar automaticamente as entregas que falham e, esgotadas as retentativas, guardar o evento em uma DLQ. | [09:15] Diego; [09:18] Diego |
| RF-14 | Reprocessar manualmente um evento da DLQ, por um endpoint restrito à role ADMIN. | [09:18] Diego; [09:36] Sofia |
| RF-15 | Registrar quem executou cada replay, para auditoria. | [09:36] Sofia |
| RF-16 | Informar em cada envio o horário do envio, para o cliente poder detectar replay attack se quiser. | [09:44] Diego |

O identificador do cliente não vem do JWT: vai no body ou no path, e a reunião não escolheu entre os dois ([09:32] Larissa). Os contratos de cada requisito estão no [FDD](FDD.md).

Limitação conhecida do RF-06: um webhook inscrito no status `PENDING` nunca dispara, porque nenhuma transição leva a esse status (`src/modules/orders/order.status.ts`) e a criação do pedido, que o coloca em `PENDING`, não passa pela mudança de status (`src/modules/orders/order.service.ts`). O detalhe está na seção 5.1 do [FDD](FDD.md).

## 7. Requisitos não funcionais

| # | Requisito | Origem |
| --- | --- | --- |
| RNF-01 | Latência: a notificação sai em menos de 10 segundos, com a outbox lida a cada 2 segundos. | [09:02] Marcos; [09:09] Diego |
| RNF-02 | Consistência: o evento é registrado na mesma transação da mudança de status. | [09:06] Diego; [09:40] Bruno |
| RNF-03 | Garantia de entrega at-least-once: o cliente pode receber o mesmo evento mais de uma vez. | [09:24] Diego |
| RNF-04 | Autenticidade e integridade: cada requisição é assinada com HMAC-SHA256, com uma secret por endpoint. | [09:20] Sofia; [09:21] Sofia; [09:22] Sofia |
| RNF-05 | Transporte: a URL do webhook precisa ser https; http é recusado. | [09:23] Sofia |
| RNF-06 | Tamanho: o payload tem limite de 64 KB, com erro caso ultrapasse. | [09:24] Larissa |
| RNF-07 | Timeout: 10 segundos por chamada; sem resposta, a entrega conta como falha. | [09:42] Diego |
| RNF-08 | Resiliência: envio inicial mais 5 retentativas com backoff exponencial, em uma janela de quase 15 horas. Os intervalos estão no [ADR-003](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md). | [09:17] Diego; [09:17] Larissa |
| RNF-09 | Ordem: garantida por pedido, enquanto houver um único worker. | [09:12] Diego; [09:13] Larissa |
| RNF-10 | Acesso: nesta fase, o CRUD de configuração aceita qualquer role autenticada; o replay exige ADMIN. | [09:36] Sofia; [09:37] Sofia |
| RNF-11 | Infraestrutura: mesmo banco e mesma stack; a outbox usa o MySQL existente, sem subir Redis. | [09:07] Diego; [09:11] Diego |
| RNF-12 | Código: o módulo segue os padrões do projeto. | [09:30] Larissa |

## 8. Decisões e trade-offs principais

Cada decisão está justificada no seu ADR; aqui fica só o que se aceitou em troca.

| Decisão | O que se aceitou | ADR |
| --- | --- | --- |
| Outbox no MySQL | Entrega assíncrona e mais uma escrita na transação da mudança de status | [ADR-001](adrs/ADR-001-outbox-no-mysql.md) |
| Worker em processo separado, com polling | Até 2 segundos de espera e um único worker por enquanto | [ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md) |
| Retry com backoff e DLQ | Desistir da entrega automática depois de quase 15 horas | [ADR-003](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md) |
| HMAC-SHA256 com secret por endpoint | Gerenciar uma secret por endpoint, com rotação e janela de 24 horas | [ADR-004](adrs/ADR-004-assinatura-hmac-sha256-com-secret-por-endpoint.md) |
| At-least-once com `X-Event-Id` | O cliente pode receber repetições e precisa deduplicar | [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) |
| Reuso dos padrões do projeto | As limitações dos componentes como estão hoje | [ADR-006](adrs/ADR-006-reuso-dos-padroes-existentes-do-projeto.md) |
| Snapshot do payload na inserção | Gravar o payload completo em cada linha da outbox | [ADR-007](adrs/ADR-007-snapshot-do-payload-na-insercao.md) |

## 9. Dependências

| Dependência | De quem | Origem |
| --- | --- | --- |
| Revisão de segurança do código de HMAC e de geração de secret antes do deploy, com pelo menos dois dias úteis reservados | Sofia | [09:46] Sofia; [09:49] Sofia |
| Documentação da integração no portal para os clientes | Marcos | [09:40] Marcos |
| Confirmação do prazo com a Atlas e aviso aos clientes | Marcos | [09:47] Marcos; [09:49] Marcos |
| Sessão de revisão do documento de design antes de começar a codar | Larissa, Bruno e Diego | [09:50] Larissa |
| O cliente verifica a assinatura do lado dele e precisa estar preparado para receber eventos repetidos | Clientes | [09:20] Sofia; [09:24] Diego |
| MySQL e base de código existentes | Interna, já existente | [09:07] Diego; [09:30] Larissa |

**Prazo e estimativa.** A Atlas pediu a entrega para o fim de novembro ([09:45] Marcos). A estimativa de Larissa é de três sprints, com a revisão de segurança incluída no fim ([09:46] Larissa; [09:47] Larissa):

| Parte | Estimativa |
| --- | --- |
| Modelagem de outbox e DLQ | 1 sprint |
| Worker e retry | 1 sprint |
| CRUD de configuração e histórico de entregas | Meia sprint |
| Integração no service de pedidos e testes ponta a ponta | Meia sprint |
| HMAC, schemas e validações | “mais um pouco” |

A reunião não disse a duração de uma sprint, então este documento não converte as três sprints em datas.

## 10. Riscos e mitigação

Esta seção traz os riscos de negócio e de prazo. Os de arquitetura e de segurança estão no [RFC](RFC.md), e os de implementação no [FDD](FDD.md).

A reunião não atribuiu probabilidade nem impacto a nenhum risco. As duas colunas são avaliação deste documento, com o motivo ao lado. O RISCO-02, o RISCO-03 e o RISCO-04 também não foram levantados como risco na reunião: este documento os formula a partir das falas citadas. O RISCO-03 parte do pedido de Sofia para reservar os dois dias úteis ([09:46] Sofia) e do lembrete dela no fim da reunião: “só não esqueçam de me agendar pra revisão de segurança antes de subir” ([09:49] Sofia).

| # | Risco | Probabilidade | Impacto | Mitigação | Origem |
| --- | --- | --- | --- | --- | --- |
| RISCO-01 | A Atlas migra para um concorrente se a feature não for entregue no prazo que pediu. | Média: o cliente já sinalizou a possibilidade. | Alto: é um dos três clientes que pediram a feature. | Marcos confirma o prazo com a Atlas e atualiza os clientes ([09:47] Marcos; [09:49] Marcos). A reunião não apresentou essas ações como resposta ao risco; a ligação é leitura deste documento. | [09:00] Marcos; [09:45] Marcos |
| RISCO-02 | A estimativa de três sprints não se confirma. Ela foi dada como chute: “Eu chuto três sprints”. | Média: as partes com estimativa já somam três sprints (1 + 1 + ½ + ½, conta deste documento), antes do “mais um pouco” de HMAC, schemas e validações. | Alto: afeta diretamente o prazo pedido pela Atlas. | Não definida na reunião. | [09:46] Larissa |
| RISCO-03 | A revisão de segurança, pré-condição do deploy, não acontece a tempo. | Baixa: a revisão foi combinada e incluída no fim das três sprints. | Alto: sem ela a feature não vai para produção. | Reservar pelo menos dois dias úteis e agendar a revisão com Sofia ([09:46] Sofia; [09:49] Sofia). | [09:46] Sofia; [09:47] Larissa |
| RISCO-04 | O cliente não é avisado quando o webhook dele está falhando, porque o alerta por email ficou para a próxima fase. | Média: indisponibilidade de cliente já aconteceu ([09:16] Diego). | Médio: as retentativas continuam por quase 15 horas ([09:17] Diego). | Não há aviso ativo nesta fase ([09:48] Larissa). O cliente pode consultar as falhas no histórico de entregas ([09:34] Marcos), e o evento que esgota as retentativas fica na DLQ para replay manual ([09:18] Diego). A ligação dessas duas funções ao risco é leitura deste documento. | [09:37] Marcos; [09:37] Larissa |

## 11. Critérios de aceitação

Esta seção traz os critérios de produto. Os critérios que um teste consegue verificar estão na seção 12 do [FDD](FDD.md).

| # | Critério | Origem |
| --- | --- | --- |
| CA-01 | Depois da mudança de status de um pedido, o envio ao webhook do cliente inscrito naquele status sai em menos de 10 segundos. | [09:02] Marcos; [09:34] Bruno |
| CA-02 | O cliente recebe só os status que escolheu para aquele webhook. | [09:33] Marcos |
| CA-03 | O cliente consegue validar que a requisição veio da plataforma e que o payload não foi adulterado. | [09:19] Sofia |
| CA-04 | O cliente consegue reconhecer um evento repetido pelo identificador do evento. | [09:25] Diego |
| CA-05 | O cliente consegue rotacionar a secret, e a antiga continua válida por 24 horas. | [09:21] Sofia |
| CA-06 | O cliente consegue consultar o histórico de entregas de um webhook. | [09:34] Marcos |
| CA-07 | Um evento que esgotou as retentativas pode ser reprocessado por um administrador, e o replay registra quem o executou. | [09:18] Diego; [09:36] Sofia |
| CA-08 | O código de HMAC e de geração de secret passou pela revisão de segurança antes do deploy. | [09:46] Sofia |

## 12. Estratégia de testes e validação

| Frente | O que cobre | Origem |
| --- | --- | --- |
| Testes ponta a ponta | A reunião não detalhou o que cobrem; foram estimados junto com a integração no service de pedidos | [09:46] Larissa |
| Testes no padrão do projeto | Os endpoints novos, com Vitest e supertest contra o MySQL real, como os testes de pedidos | Proposta deste documento, seguindo `tests/orders.test.ts`, `tests/setup.ts` e `package.json` |
| Critérios técnicos | Os 16 critérios da seção 12 do [FDD](FDD.md), usados como roteiro para os testes | Proposta deste documento |
| Revisão de segurança | O código de HMAC e de geração de secret, revisado por Sofia antes do deploy | [09:46] Sofia |
| Revisão do design | O documento de design, revisado por Larissa, Bruno e Diego antes de começar a codar | [09:50] Larissa |

Sobre testes, a reunião só falou em testes ponta a ponta. As duas linhas marcadas como proposta são deste documento. A reunião também não definiu validação com os clientes, como piloto ou homologação, antes da entrega.
