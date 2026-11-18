# Inventário da transcrição

Este arquivo registra, em ordem cronológica, cada ponto substantivo da reunião técnica sobre o Sistema de Webhooks de Notificação de Pedidos (`TRANSCRICAO.md`). É o gabarito usado para redigir o PRD, o RFC, o FDD, os ADRs e o Tracker: decisão, requisito, restrição, alternativa descartada ou item adiado só entra em um documento se estiver aqui ou tiver origem no código. Falas de apoio citadas nos documentos, como justificativas e exemplos, vêm direto da transcrição.

## Como ler

- Cada linha traz o minuto, o falante e uma citação literal da fala, entre aspas curvas. Aspas retas dentro da citação pertencem à própria transcrição.
- Caminhos e nomes de arquivo que aparecem dentro das citações são os ditos na reunião; alguns ainda não existem no repositório.
- Quando o ponto cita uma segunda fala, ela aparece como `Nome [hh:mm]`.
- O Destaque E, no fim do arquivo, é o único trecho que confronta as falas com o código: lista os pontos em que a reunião afirma algo que o código não confirma.

Classes usadas:

| Sigla | Significado |
| --- | --- |
| CTX | Informação de contexto |
| RF | Requisito funcional |
| RNF | Requisito não funcional |
| RES | Restrição |
| DEC | Decisão fechada |
| ALT | Alternativa descartada |
| ADI | Item adiado para fase futura |
| QA | Questão em aberto |
| DET | Detalhe técnico secundário |
| PRZ | Prazo |

## Tabela cronológica

| # | Quando | Quem | Classe | Ponto | Citação |
| --- | --- | --- | --- | --- | --- |
| I-01 | 09:00 | Marcos | CTX | Demanda formal de três clientes B2B | “pedido formal de três clientes B2B: Atlas Comercial, MaxDistribuição e Nova Cargo” |
| I-02 | 09:00 | Marcos | CTX | Problema atual: clientes fazem polling na API | “Hoje eles ficam batendo no GET /orders de tempos em tempos” |
| I-03 | 09:00 | Marcos | CTX | Risco de negócio: perda da Atlas | “se a gente não entregar isso até fim do trimestre, eles podem migrar pro nosso concorrente” |
| I-04 | 09:02 | Marcos | RNF | Latência aceita pelos clientes: abaixo de 10 s | “qualquer coisa abaixo de 10 segundos já é "tempo real"” |
| I-05 | 09:02 | Marcos | RES | Só webhooks de saída | “Só saindo da gente pra eles.” |
| I-06 | 09:04 | Bruno | ALT | Envio síncrono dentro do service de orders. Motivo 1: cliente lento trava outras mudanças de status | “qualquer cliente lento vai travar mudança de status pra outros pedidos” |
| I-07 | 09:04 | Bruno | ALT | Envio síncrono. Motivo 2: cliente fora do ar forçaria rollback do status | “dá rollback na mudança de status? Não dá.” |
| I-08 | 09:06 | Diego | ALT | Confirma o descarte do síncrono | “Síncrono está fora de questão.” |
| I-09 | 09:06 | Diego | DEC | Outbox, e não fila (ver correção COR-4): evento gravado na mesma transação da mudança de status | “dentro da mesma transação SQL que atualiza orders e order_status_history, a gente também insere uma linha numa tabela tipo webhook_outbox” |
| I-10 | 09:06 | Diego | DEC | Garantia de atomicidade do outbox | “se a transação principal commitou, o evento foi registrado, e se ela deu rollback, o evento some junto” |
| I-11 | 09:07 | Larissa | ALT | Redis Streams. Motivo: exige infraestrutura nova | “a gente acabaria precisando subir mais infra” |
| I-12 | 09:07 | Diego | ALT | Redis. Motivo: time pequeno | “Subir Redis Cluster pra isso é overengineering.” |
| I-13 | 09:08 | Diego | DET | Status e índices da outbox | “índice no campo de status (pendente, processando, falhou, entregue) e em created_at” |
| I-14 | 09:08 | Diego | DET | Leitura em lote, sem número definido | “Worker lê só os pendentes em batch pequeno” |
| I-15 | 09:08 | Diego | ADI | Arquivamento de linhas entregues | “Linhas entregues a gente arquiva depois de 30 dias ou assim, fora do escopo dessa feature.” |
| I-16 | 09:08 | Larissa | DEC | Outbox no MySQL | “Tá decidido então: outbox em MySQL.” |
| I-17 | 09:09 | Diego | DEC | Polling a cada 2 s, mais antigos primeiro | “A cada 2 segundos, busca os eventos pendentes mais antigos” |
| I-18 | 09:09 | Diego | ALT | Trigger do banco, sugerida por Bruno [09:09]. Motivo: MySQL não avisa processo externo | “MySQL não tem listener nativo tipo o NOTIFY/LISTEN do Postgres.” |
| I-19 | 09:10 | Marcos | CTX | Aceite do PM para os 2 s | “2 segundos serve, perfeito.” |
| I-20 | 09:10 | Larissa | DEC | Worker em polling de 2 s, com a latência aceita como trade-off | “Worker em polling, 2s. A latência mínima vai ser 2 segundos no pior caso. Aceitamos.” |
| I-21 | 09:11 | Diego | DEC | Worker em processo separado da API | “o worker tem que rodar como processo separado, não dentro da mesma instância da API” |
| I-22 | 09:11 | Diego | DEC | Motivo do processo separado | “Senão se a API reinicia, perde o worker.” |
| I-23 | 09:11 | Larissa | DET | Entry point e script novos | “criar um src/worker.ts e um script "npm run worker"” |
| I-24 | 09:11 | Bruno | (corrigida) | Mesmo Prisma client: ver correção COR-2 | “usar o mesmo Prisma client” |
| I-25 | 09:11 | Diego | RES | Mesmo banco e mesma stack | “Sim, mesmo banco, mesma stack.” |
| I-26 | 09:12 | Diego | DEC | Um único worker; ordem por pedido | “Por enquanto, single-worker e ordering implícita por order_id.” |
| I-27 | 09:13 | Diego | ADI | Escala para vários workers | “Aí dá pra particionar por order_id, ou usar lock pessimista. Mas isso é problema do futuro, não agora.” |
| I-28 | 09:13 | Larissa | RES | Limitação conhecida: sem ordem global | “Não é garantia de ordering global, só por order_id e enquanto for single-worker.” |
| I-29 | 09:14 | Marcos | CTX | Clientes não pediram ordem global | “Os clientes nunca pediram garantia de ordering global” |
| I-30 | 09:15 | Diego | DEC | Backoff exponencial, teto de tentativas, DLQ | “depois de um teto de tentativas considera falha permanente e move pra DLQ” |
| I-31 | 09:15 | Diego | ALT | Retry indefinido. Motivo: evento nunca termina | “evento ficar pendurado pra sempre se o cliente sumiu” |
| I-32 | 09:15 | Diego | (refinada) | Janela estimada: ver correção COR-3 | “Cinco já dá pra cobrir uma janela de até 12 ou 24 horas.” |
| I-33 | 09:16 | Diego | ALT | 3 tentativas, sugestão de Bruno [09:16]. Motivo: desiste em 30 min | “a gente retentaria três vezes em 30 minutos e mataria” |
| I-34 | 09:16 | Diego | CTX | Precedente que justifica mais tentativas | “Já tinha cliente nosso com indisponibilidade de duas horas em manutenção planejada.” |
| I-35 | 09:17 | Diego | DEC | Progressão do backoff | “1 minuto, 5 minutos, 30 minutos, 2 horas, 12 horas. Total de quase 15 horas entre primeira falha e última tentativa.” |
| I-36 | 09:17 | Marcos | CTX | Aceite do PM para a janela de 15 h | “Se um cliente meu cair por 15 horas, ele já tá com problema sério dele. Acho aceitável.” |
| I-37 | 09:17 | Larissa | DEC | 5 tentativas com o backoff acima (leitura adotada: ver ambiguidade AMB-1) | “Decidido: 5 tentativas, backoff 1m/5m/30m/2h/12h.” |
| I-38 | 09:17 | Larissa | ALT | DLQ como marcação na própria outbox | “marca como "failed" na própria outbox” |
| I-39 | 09:18 | Diego | DEC | DLQ em tabela separada | “uma tabela webhook_dead_letter separada, com a payload, motivo da falha e timestamp” |
| I-40 | 09:18 | Diego | DEC | Motivo da tabela separada | “Mais limpa a leitura da outbox principal” |
| I-41 | 09:18 | Diego | RF | Replay manual da DLQ por endpoint admin | “POST /admin/webhooks/dead-letter/:id/replay. Recoloca na outbox como pendente.” |
| I-42 | 09:19 | Sofia | RNF | Cliente precisa validar origem e integridade | “O cliente tem que conseguir validar que a requisição veio realmente da gente, e que ninguém adulterou o payload no meio.” |
| I-43 | 09:20 | Sofia | DEC | Assinatura HMAC em header | “manda a assinatura num header tipo X-Signature” |
| I-44 | 09:20 | Sofia | DEC | Algoritmo SHA-256 | “HMAC-SHA256 é o padrão de mercado” |
| I-45 | 09:21 | Sofia | DEC | Secret única por endpoint | “cada endpoint de webhook do cliente tem que ter uma secret única” |
| I-46 | 09:21 | Sofia | ALT | Secret global da plataforma | “Senão se vaza uma, vaza tudo.” |
| I-47 | 09:21 | Bruno | DET | Campos da configuração, confirmados por Sofia | “url + secret + customer_id + estado ativo” |
| I-48 | 09:21 | Sofia | RF | Rotação de secret pela API | “Endpoint pro cliente conseguir pedir nova secret pela API.” |
| I-49 | 09:21 | Sofia | RF | Secret antiga vale 24 h após a rotação | “a antiga fica válida por 24 horas em paralelo” |
| I-50 | 09:22 | Diego | CTX | Precedente de vazamento de secret | “A gente já teve cliente que vazou secret em log de aplicação dele uma vez.” |
| I-51 | 09:22 | Sofia | DEC | Fechamento da autenticação | “Decidido: HMAC-SHA256 sobre o corpo do request, secret por endpoint, suporte a rotação com grace period de 24h.” |
| I-52 | 09:23 | Sofia | RF | URL precisa ser https | “Se o cliente cadastrar http, recusamos com erro de validação.” |
| I-53 | 09:23 | Sofia | DET | https é validação, não decisão arquitetural | “é só uma validação no schema Zod” |
| I-54 | 09:23 | Sofia | ALT | Truncar payload grande | “Trunca? Erra? Eu sou a favor de erra.” |
| I-55 | 09:24 | Larissa | RNF | Limite de 64 KB por payload, teto sugerido por Diego [09:24] | “64KB de limite, erro caso ultrapasse.” |
| I-56 | 09:24 | Diego | DEC | Garantia at-least-once | “a gente vai garantir at-least-once” |
| I-57 | 09:25 | Diego | DEC | Identificador único do evento | “X-Event-Id, com um UUID gerado quando o evento entra na outbox” |
| I-58 | 09:25 | Sofia | CTX | Trade-off: deduplicação fica com o cliente | “Isso joga responsabilidade pro cliente.” |
| I-59 | 09:25 | Diego | ALT | Exactly-once. Motivo: complexidade | “Garantir exactly-once exigiria coordenação dos dois lados e fica muito mais complexo.” |
| I-60 | 09:26 | Marcos | CTX | Dependência: documentação no portal | “Eu posso documentar isso bem destacado no portal de desenvolvedor pros clientes” |
| I-61 | 09:26 | Larissa | DEC | Fechamento da garantia de entrega | “At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão.” |
| I-62 | 09:27 | Bruno | DEC | Módulo novo no padrão existente | “Vou propor uma pasta src/modules/webhooks com toda a estrutura.” |
| I-63 | 09:28 | Bruno | DET | Arquivo da lógica do worker, nome não fechado | “src/modules/webhooks/webhook.worker.ts ou webhook.processor.ts” |
| I-64 | 09:28 | Bruno | DEC | Códigos de erro no padrão do projeto | “Códigos tipo WEBHOOK_NOT_FOUND, WEBHOOK_INVALID_URL, WEBHOOK_SECRET_REQUIRED, etc.” |
| I-65 | 09:29 | Larissa | DEC | Prefixo dos códigos | “Prefixo WEBHOOK_ pra tudo do módulo.” |
| I-66 | 09:29 | Bruno | RES | Sem biblioteca nova de log | “Não vamos botar nada novo.” |
| I-67 | 09:30 | Bruno | DEC | Instância própria do Prisma no worker (corrige I-24) | “Separado. PrismaClient é por processo.” |
| I-68 | 09:30 | Larissa | DEC | Reuso dos padrões do projeto | “Decisão: reuso máximo do que já existe.” |
| I-69 | 09:31 | Marcos | RF | Cadastro de webhook | “O cliente precisa cadastrar webhook. Endpoint POST.” |
| I-70 | 09:31 | Marcos | RF | Secret gerada pelo sistema | “secret é gerada pela gente e devolvida na criação” |
| I-71 | 09:31 | Marcos | RF | Lista de status por webhook | “Lista de status que ele quer receber.” |
| I-72 | 09:31 | Marcos | (corrigida) | customer_id pelo JWT: ver correção COR-1 | “Customer_id implícito do JWT.” |
| I-73 | 09:32 | Marcos | DEC | Cliente usa a API, não um painel | “Pela nossa API direto, autenticado com JWT do nosso sistema.” |
| I-74 | 09:32 | Larissa | RF | customer_id explícito na requisição (corrige I-72) | “o customer_id é passado no body ou no path. Não vem do JWT.” |
| I-75 | 09:33 | Bruno | RF | Editar, remover e listar webhooks | “PATCH pra editar, DELETE pra remover, GET pra listar os webhooks de um customer.” |
| I-76 | 09:33 | Marcos | RF | Filtro de eventos por status | “Filtro de eventos é uma lista dos status que o webhook quer ouvir.” |
| I-77 | 09:34 | Bruno | DEC | Filtro aplicado na inserção da outbox | “Na inserção. Se nenhum webhook do customer quer aquele status, nem insere.” |
| I-78 | 09:34 | Bruno | ALT | Filtrar na hora do envio. Motivo do descarte | “Economiza linha na tabela.” |
| I-79 | 09:34 | Marcos | RF | Histórico de entregas (o número é exemplo: ver ambiguidade AMB-3) | “os últimos 100 webhooks que vocês mandaram pra mim, sucesso/falha, payload, response, tempo de resposta” |
| I-80 | 09:34 | Marcos | RF | Endpoint do histórico | “GET /webhooks/:id/deliveries” |
| I-81 | 09:36 | Sofia | RF | Replay só para ADMIN | “Mexer em fila de entrega de notificação não é coisa de operador.” |
| I-82 | 09:36 | Sofia | RF | Auditoria do replay | “o endpoint de admin tem que logar quem fez o replay, pra auditoria” |
| I-83 | 09:36 | Larissa | DEC | Reuso do `requireRole` | “role ADMIN obrigatório no replay e a gente reaproveita o requireRole que já existe” |
| I-84 | 09:37 | Sofia | DEC + ADI | CRUD aberto a qualquer role autenticada; endurecer depois | “Por enquanto sim. Mais pra frente a gente pode endurecer.” |
| I-85 | 09:37 | Larissa | ADI | Alerta por email, pedido por Marcos [09:37] (versão final: ver correção COR-5) | “Email tá fora de escopo dessa fase. Talvez próxima fase, depois que a gente medir o impacto.” |
| I-86 | 09:39 | Diego | QA | Rate limiting de saída, levantado por Diego [09:38] | “A gente observa e implementa se virar problema. Mas vale registrar como ponto em aberto.” |
| I-87 | 09:39 | Larissa | QA | Encaminhamento do rate limiting | “Fica como "observar e decidir depois".” |
| I-88 | 09:40 | Larissa | ADI | Dashboard visual, pedido por Marcos [09:39] | “Só endpoints. Painel é projeto separado do time de frontend.” |
| I-89 | 09:40 | Marcos | CTX | Dependência: guia de integração no portal | “Eu documento no portal pros clientes saberem como integrar via API.” |
| I-90 | 09:40 | Bruno | DEC | Falha ao gravar na outbox desfaz a mudança de status | “Se a outbox falhar de inserir, rollback. Não pode ter caso de status mudar e evento não sair.” |
| I-91 | 09:41 | Bruno | DET | Função de publicação que recebe o `tx` | “publishWebhookEvent(tx, order, fromStatus, toStatus)” |
| I-92 | 09:41 | Diego | ALT | Injetar o repository de webhook no OrderService | “Não precisa injetar repository inteiro.” |
| I-93 | 09:42 | Diego | RNF | Timeout de 10 s por chamada | “Cliente lento que não responde em 10s a gente trata como falha e marca pra retry.” |
| I-94 | 09:43 | Diego | DET | Formato do payload | “event_type tipo "order.status_changed"” |
| I-95 | 09:43 | Diego | DET | Payload sem itens do pedido | “Não manda items pra não inflar.” |
| I-96 | 09:44 | Diego | DET | Headers do envio | “X-Event-Id com o UUID, X-Signature com o HMAC, X-Timestamp com o timestamp do envio (pra cliente conseguir detectar replay attack se quiser), Content-Type application/json” |
| I-97 | 09:44 | Sofia | DET | Header adicional, aceito por Diego [09:45] | “Adiciona um X-Webhook-Id também” |
| I-98 | 09:45 | Marcos | PRZ | Data pedida pelo cliente | “A Atlas quer pra fim de novembro.” |
| I-99 | 09:46 | Larissa | PRZ | Estimativa, com a divisão por sprint | “Modelagem de outbox e DLQ é uma sprint. Worker e retry é uma sprint. CRUD de configuração e deliveries é meio sprint. Integração no order.service e testes ponta a ponta é mais meio. HMAC, schemas, validações, mais um pouco. Eu chuto três sprints incluindo revisão da Sofia.” |
| I-100 | 09:46 | Sofia | RES | Revisão de segurança antes do deploy | “Reservem pelo menos dois dias úteis pra eu revisar o código de segurança antes do deploy.” |
| I-101 | 09:47 | Larissa | PRZ | Prazo fechado | “Três sprints com a revisão da Sofia incluída no fim.” |
| I-102 | 09:48 | Larissa | — | Resumo final da reunião (marco) | “Email como fallback fica pra próxima fase.” |
| I-103 | 09:50 | Larissa | CTX | Próximo passo: documento de design | “Eu vou abrir o doc de design da feature” |
| I-104 | 09:51 | Larissa | DEC | Id da outbox em UUID; descarta auto incremental | “UUID, segue o padrão do resto do projeto. Tudo é uuid.” |
| I-105 | 09:52 | Larissa | DEC | Payload gravado como snapshot na inserção | “Eu prefiro renderizado já, na hora da inserção.” |
| I-106 | 09:52 | Larissa | DEC | Motivo do snapshot | “Se o pedido mudar depois, o evento ainda reflete o estado de quando o status mudou.” |
| I-107 | 09:52 | Bruno | DEC | Fechamento do snapshot | “Beleza, snapshot. Decidido.” |

## Destaque A — Falas corrigidas

Vale sempre a versão final.

| # | Versão inicial | Versão final | Efeito nos documentos |
| --- | --- | --- | --- |
| COR-1 | [09:31] Marcos: “Customer_id implícito do JWT.” | [09:32] Larissa: “o customer_id é passado no body ou no path. Não vem do JWT.” | Nenhum documento diz que o cliente é identificado pelo token. |
| COR-2 | [09:11] Bruno: “usar o mesmo Prisma client” | [09:30] Bruno: “Separado. PrismaClient é por processo.” | Mesmo banco, instância própria no worker. |
| COR-3 | [09:15] Diego: “Cinco já dá pra cobrir uma janela de até 12 ou 24 horas.” | [09:17] Diego: “Total de quase 15 horas entre primeira falha e última tentativa.” | A janela de retry é de cerca de 15 h. |
| COR-4 | [09:03] Larissa: “faz algum tipo de fila/outbox?” | [09:06] Diego: “eu nem chamaria de "fila"” | Os documentos falam em outbox, não em fila. |
| COR-5 | [09:37] Larissa: “Talvez próxima fase, depois que a gente medir o impacto.” | [09:48] Larissa: “Email como fallback fica pra próxima fase.” | Email é item adiado; o resumo final é mais firme que a fala original. |

## Destaque B — Decidido depois do resumo final

O resumo final é de [09:48]. Marcos e Sofia saíram em [09:50]; só Larissa, Diego e Bruno participaram destas decisões.

- **Id da outbox em UUID** (I-104). Alternativa descartada: auto incremental, perguntado em [09:51] Diego: “prefere id auto incremental ou UUID?”
- **Payload em snapshot na inserção** (I-105 a I-107). Alternativa descartada: renderizar no envio, perguntado em [09:51] Bruno: “guarda só order_id e renderiza na hora do envio”

## Destaque C — Decidido antes do resumo, mas ausente dele

O resumo de [09:48] não menciona estes itens, que correm o mesmo risco de esquecimento. A lista traz os de maior peso para os documentos e não é exaustiva:

- Worker único e ordem só por pedido (I-26, I-28).
- URL https (I-52), limite de 64 KB (I-55) e timeout de 10 s (I-93).
- Filtro por status na inserção (I-77).
- Histórico de entregas (I-79, I-80) e auditoria do replay (I-82).
- `customer_id` fora do JWT (I-74) e secret gerada pelo sistema (I-70).
- Formato do payload e headers além do `X-Event-Id` (I-94 a I-97).
- Dois dias úteis de revisão de segurança (I-100).
- Arquivamento adiado (I-15).

## Destaque D — Ambiguidades e leitura adotada

| # | Ambiguidade | Leitura adotada |
| --- | --- | --- |
| AMB-1 | "5 tentativas" (I-37) pode ser 5 envios no total ou 5 reenvios. | Envio inicial mais 5 retentativas. Evidência: 3 tentativas dariam “a gente retentaria três vezes em 30 minutos e mataria” ([09:16] Diego), e os três primeiros intervalos somam 36 min; os cinco intervalos somam 14h36, o “Total de quase 15 horas entre primeira falha e última tentativa” ([09:17] Diego). |
| AMB-2 | O `customer_id` vai “no body ou no path” (I-74); a reunião não escolheu. | Questão em aberto. |
| AMB-3 | “os últimos 100 webhooks” (I-79) pode ser regra ou exemplo. | Exemplo. O histórico de entregas segue a paginação que o projeto já usa; a retenção dos registros não foi discutida e fica fora dos documentos. |
| AMB-4 | A duração da sprint nunca foi dita (I-98 a I-101). | Os documentos citam "três sprints" e "fim de novembro" sem converter um no outro. |
| AMB-5 | O status "falhou" da outbox (I-13) convive com a DLQ em tabela separada (I-39). | Questão em aberto, a decidir no FDD. |
| AMB-6 | `WEBHOOK_SECRET_REQUIRED` (I-64) foi citado antes de se decidir que a secret é gerada pelo sistema (I-70). | Questão em aberto, a decidir no FDD. |
| AMB-7 | O nome do arquivo com a lógica do worker ficou entre duas opções (I-63). | Questão em aberto, a decidir no FDD. |
| AMB-8 | Há duas referências de prazo para a mesma feature e o mesmo cliente: “até fim do trimestre” (I-03) e “fim de novembro” (I-98). | Vale a fala mais recente: o prazo pedido é fim de novembro, inclusive quando os documentos tratam do risco de perder a Atlas. "Fim do trimestre" não é usado como data. |
| AMB-9 | “A latência mínima vai ser 2 segundos no pior caso.” (I-20) junta "mínima" e "pior caso". | O evento espera até 2 segundos pelo ciclo de leitura do worker. |

Ruído desconsiderado: em [09:27] Larissa: “Estamos a uma hora de reunião”, embora o relógio da transcrição marque 27 minutos.

## Destaque E — Divergências entre a reunião e o código

Pontos em que uma fala da reunião não bate com o que o código tem hoje, e como cada um é registrado nos documentos.

| # | O que a reunião diz | O que o código tem | Registro adotado |
| --- | --- | --- | --- |
| DIV-1 | [09:04] Bruno: “decrementa stock_quantity dos produtos do pedido” | O estoque muda em 3 das 7 transições permitidas: débito em PENDING→PAID e reposição em PAID→CANCELLED e PROCESSING→CANCELLED (`src/modules/orders/order.status.ts`). O campo se chama `stockQuantity` (`prisma/schema.prisma`). | Os documentos descrevem o código como ele é. |
| DIV-2 | [09:31] Marcos: “Customer_id implícito do JWT.” [09:32] Marcos: “A gente tem usuários que representam o cliente.” | `User` não tem relação com `Customer` (`prisma/schema.prisma`), e o JWT leva só id, email e role (`src/middlewares/auth.middleware.ts`). | Vale a correção COR-1. Pelo código, qualquer usuário autenticado alcança webhooks de qualquer cliente. Sobre restrição de acesso, a reunião só tratou de role: aceitou qualquer role autenticada no CRUD nesta fase ([09:37] Sofia: “Por enquanto sim.”). |
| DIV-3 | [09:36] Larissa: “role ADMIN obrigatório no replay e a gente reaproveita o requireRole que já existe” | O cadastro de usuário é aberto e aceita a role `ADMIN` no corpo (`src/modules/auth/auth.routes.ts`, `src/modules/auth/auth.schemas.ts`). | Risco, com fonte no código. |
| DIV-4 | [09:28] Bruno: “WEBHOOK_INVALID_URL” [09:23] Sofia: “é só uma validação no schema Zod” | Toda falha de schema Zod sai como `VALIDATION_ERROR` (`src/middlewares/validate.middleware.ts`, `src/shared/errors/http-errors.ts`). | Questão em aberto, a decidir no FDD. |
| DIV-5 | [09:51] Larissa: “Tudo é uuid.” | `OrderNumberSequence` usa id inteiro (`prisma/schema.prisma`). | Os documentos dizem "models de domínio". |
| DIV-6 | As rotas são ditas sem prefixo, como em [09:34] Marcos: “GET /webhooks/:id/deliveries” | Todas as rotas dos módulos são montadas em `/api/v1`; só `/health` fica fora (`src/app.ts`). | Os contratos do FDD usam o prefixo; as citações de fala mantêm a rota como foi dita. |
| DIV-7 | [09:27] Bruno: “Cada domínio é um módulo em src/modules com controller, service, repository, routes e schemas.” | O módulo `auth` não tem repository (`src/modules/auth/`). Um módulo novo é registrado à mão em `src/app.ts` e em `src/routes/index.ts`. | Os documentos descrevem o código como ele é; o registro manual entra no FDD. |
| DIV-8 | [09:43] Diego: “order_id, order_number, from_status, to_status, customer_id” | A API REST usa camelCase, como `customerId` e `pageSize` (`src/modules/orders/order.schemas.ts`). | Payload do webhook em snake_case, como dito; endpoints de configuração em camelCase, como o resto da API. |
| DIV-9 | [09:33] Marcos: “Filtro de eventos é uma lista dos status que o webhook quer ouvir.” | Nenhuma transição leva a `PENDING` (`src/modules/orders/order.status.ts`), e a criação do pedido grava o histórico inicial sem passar pelo `changeStatus` (`src/modules/orders/order.service.ts`). | Limitação conhecida: um webhook inscrito em `PENDING` nunca dispara. |
