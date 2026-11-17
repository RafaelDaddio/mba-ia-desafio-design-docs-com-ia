# ADR-002: Worker em processo separado, lendo a outbox em polling

## Status

Aceito. Decisão tomada na reunião técnica do Sistema de Webhooks de Notificação de Pedidos (`TRANSCRICAO.md`). Participaram da decisão: Larissa, Diego e Bruno, com aceite de Marcos para a latência.

## Contexto

Os eventos ficam gravados na outbox ([ADR-001](ADR-001-outbox-no-mysql.md)) e alguém precisa lê-los e fazer as chamadas HTTP. Para os clientes, qualquer atraso abaixo de 10 segundos já conta como tempo real ([09:02] Marcos).

O MySQL não oferece um aviso nativo a processos externos quando uma linha é inserida ([09:09] Diego). A aplicação tem hoje um único entry point, `src/server.ts`, e os scripts `dev` e `start` de `package.json` iniciam apenas esse processo.

## Decisão

Um worker lê a outbox em polling: a cada 2 segundos busca os eventos pendentes mais antigos, processa e marca ([09:09] Diego; [09:10] Larissa).

O worker roda como processo separado da API, com entry point próprio nos moldes de `src/server.ts`, usando o mesmo banco e a mesma stack ([09:11] Diego; [09:11] Larissa). Por ser outro processo Node, ele abre a sua própria instância do `PrismaClient`, com a mesma `DATABASE_URL` ([09:30] Bruno).

Há um único worker em execução ([09:12] Diego).

## Alternativas Consideradas

### Trigger do banco para reagir na hora

Sugerida para tornar a entrega mais reativa ([09:09] Bruno). Descartada porque o MySQL não tem um listener nativo como o NOTIFY/LISTEN do Postgres: a trigger só executa SQL e não avisa um processo externo. Avisar o worker exigiria um improviso, como escrever em arquivo ou chamar um endpoint ([09:09] Diego).

### Worker dentro do processo da API

Descartado porque o worker cairia junto a cada reinício da API ([09:11] Diego: “Senão se a API reinicia, perde o worker.”).

## Consequências

### Positivas

- O intervalo de 2 segundos atende com folga o limite de 10 segundos pedido pelos clientes ([09:09] Diego), e foi aceito pelo produto ([09:10] Marcos).
- O worker não é perdido quando a API reinicia, motivo dado para separar os processos ([09:11] Diego).
- Com um único worker processando por ordem de criação, o cliente recebe em ordem os eventos de um mesmo pedido ([09:12] Diego).

### Negativas

- Todo evento espera até 2 segundos pelo próximo ciclo de leitura, atraso aceito na reunião ([09:10] Larissa).
- Não há garantia de ordem global. A ordem vale apenas por pedido, e apenas enquanto houver um único worker ([09:13] Larissa). Os clientes não pediram ordem global ([09:14] Marcos).
- Escalar para vários workers quebra essa garantia ([09:12] Diego). As saídas citadas foram particionar por pedido ou usar lock pessimista, e o assunto foi adiado ([09:13] Diego).
- O worker depende de um entry point e de um script `npm run worker` que ainda não existem ([09:11] Larissa): os scripts `dev` e `start` de `package.json` iniciam só o servidor.

### Trade-off

Aceitamos até 2 segundos de atraso e a limitação a um único worker, sem ordem global e sem escala horizontal por enquanto, para cumprir o limite de 10 segundos com o mesmo banco e a mesma stack.

## Referências

- Transcrição: [09:02] Marcos; [09:09] Diego; [09:09] Bruno; [09:10] Marcos; [09:10] Larissa; [09:11] Diego; [09:11] Larissa; [09:12] Diego; [09:13] Diego; [09:13] Larissa; [09:14] Marcos; [09:30] Bruno.
- Código: `src/server.ts` (entry point atual); `package.json` (scripts).
- ADRs relacionados: [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-006](ADR-006-reuso-dos-padroes-existentes-do-projeto.md).
