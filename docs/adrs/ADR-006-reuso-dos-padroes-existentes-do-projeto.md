# ADR-006: Reuso dos padrões existentes do projeto

## Status

Aceito. Decisão tomada na reunião técnica do Sistema de Webhooks de Notificação de Pedidos (`TRANSCRICAO.md`). Participaram da decisão: Bruno, que a propôs, Larissa e Diego. A exigência de role ADMIN no replay veio de Sofia ([09:36] Sofia). O id da outbox em UUID foi confirmado depois do resumo final, com Marcos e Sofia já fora da chamada ([09:51] Larissa).

## Contexto

O projeto organiza cada domínio como um módulo em `src/modules/` (`auth`, `users`, `customers`, `products` e `orders`), e Bruno abriu o bloco de estrutura afirmando: “A gente tem um padrão claro na codebase.” ([09:27] Bruno). Além da estrutura de módulo, ele listou o que já existe e a feature vai usar: as classes de erro com código ([09:28] Bruno), o logger Pino e o middleware de erro centralizado ([09:29] Bruno).

## Decisão

O módulo de webhooks reaproveita ao máximo o que já existe ([09:30] Larissa: “Decisão: reuso máximo do que já existe.”).

| Padrão | Onde está no código | Como a feature usa | Origem |
| --- | --- | --- | --- |
| Estrutura de módulo | `src/modules/orders/` (`order.controller.ts`, `order.service.ts`, `order.repository.ts`, `order.routes.ts`, `order.schemas.ts`) | Um módulo `webhooks` dentro de `src/modules/`, com a mesma divisão de arquivos | [09:27] Bruno |
| Classes de erro | `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts` | Erros do módulo derivam de `AppError` | [09:28] Bruno |
| Códigos de erro | `INSUFFICIENT_STOCK` e `INVALID_STATUS_TRANSITION` em `src/shared/errors/http-errors.ts` | Mesmo formato, com o prefixo `WEBHOOK_` em todos os códigos do módulo | [09:28] Bruno; [09:29] Larissa |
| Logger | `src/shared/logger/index.ts` (Pino) | Mesmo logger, sem biblioteca nova | [09:29] Bruno |
| Tratamento de erro | `src/middlewares/error.middleware.ts` | Usado como está | [09:29] Bruno |
| Validação | Schemas Zod, como em `src/modules/orders/order.schemas.ts` | Schemas Zod para as entradas do módulo | [09:30] Larissa |
| Controle de acesso | `requireRole` em `src/middlewares/auth.middleware.ts` | Exige a role ADMIN no replay da DLQ | [09:36] Larissa |
| Identificadores | `@default(uuid())` nos models de domínio de `prisma/schema.prisma` | Id da outbox em UUID | [09:51] Larissa |

## Alternativas Consideradas

### Id auto incremental na outbox

Levantada como dúvida de modelagem ([09:51] Diego). Descartada para seguir o padrão do projeto ([09:51] Larissa: “UUID, segue o padrão do resto do projeto. Tudo é uuid.”).

### Convenções próprias para o módulo (plausível, não discutida na reunião)

Dar ao módulo de webhooks a sua própria estrutura, o seu formato de erro ou a sua biblioteca de log. Permitiria contornar as limitações de código de erro e de logger listadas abaixo, ao custo de um segundo padrão na mesma base. A reunião foi na direção oposta ([09:30] Larissa).

## Consequências

### Positivas

- O error middleware já responde a qualquer `AppError` no formato `{ error: { code, message, details } }` (`src/middlewares/error.middleware.ts`), então os erros do módulo são tratados sem mudança nele ([09:29] Bruno).
- Nenhuma biblioteca nova de log entra no projeto ([09:29] Bruno).
- A regra de acesso do replay usa um middleware que já existe e já é usado em `src/modules/users/user.routes.ts` ([09:36] Larissa).

### Negativas

Todas vêm do estado atual do código:

- `NotFoundError`, `ValidationError`, `UnauthorizedError` e `ForbiddenError` têm código fixo (`NOT_FOUND`, `VALIDATION_ERROR`, `UNAUTHORIZED` e `FORBIDDEN`) em `src/shared/errors/http-errors.ts`. As duas últimas são as que `authenticate` e `requireRole` lançam (`src/middlewares/auth.middleware.ts`), então as recusas de autenticação e de role saem sem o prefixo `WEBHOOK_`. Os demais códigos `WEBHOOK_*` dependem de classes novas para o módulo ou das classes que aceitam código customizado (`BadRequestError`, `ConflictError`, `UnprocessableEntityError`).
- Toda falha de schema Zod passa por `src/middlewares/validate.middleware.ts` e sai como `VALIDATION_ERROR`. A reunião quer que a URL http seja recusada no schema Zod ([09:23] Sofia) e também cita o código `WEBHOOK_INVALID_URL` ([09:28] Bruno). Qual código essa recusa devolve fica como questão em aberto.
- O logger de `src/shared/logger/index.ts` tem o nome de serviço fixo em `order-management-api`, e a sua lista de campos mascarados (`redactPaths`) cobre senha, token e os headers de autorização e cookie, sem nenhum campo de secret.
- No estado atual do código, a exigência de role ADMIN no replay pode ser contornada por autocadastro. O caminho, arquivo por arquivo:
  - A rota de cadastro, `POST /api/v1/auth/register`, não passa pelo middleware `authenticate` (`src/modules/auth/auth.routes.ts`), e o `registerSchema` aceita o campo `role` com o valor `ADMIN` (`src/modules/auth/auth.schemas.ts`).
  - O cadastro grava a role recebida (`src/modules/users/user.service.ts`), e o login coloca essa role no JWT (`src/modules/auth/auth.service.ts`).
  - `requireRole` compara a role exigida somente com a que veio no JWT (`src/middlewares/auth.middleware.ts`).

  Quem se cadastra informando `ADMIN` passa pelo `requireRole` e alcança o endpoint de replay.
- A afirmação “Tudo é uuid” ([09:51] Larissa) vale para os models de domínio; `OrderNumberSequence`, em `prisma/schema.prisma`, usa id inteiro.

### Trade-off

Aceitamos as limitações de padrão dos componentes como estão hoje (códigos fixos em parte das classes de erro e um logger com configuração única) para obter um módulo igual aos outros da base, sem biblioteca nova de log.

O contorno da role ADMIN por autocadastro não faz parte desse aceite: a reunião não tratou do cadastro de usuários.

## Referências

- Transcrição: [09:23] Sofia; [09:27] Bruno; [09:28] Bruno; [09:29] Bruno; [09:29] Larissa; [09:30] Larissa; [09:36] Sofia; [09:36] Larissa; [09:51] Diego; [09:51] Larissa.
- Código: `src/modules/`; `src/modules/orders/`; `src/modules/orders/order.schemas.ts`; `src/shared/errors/app-error.ts`; `src/shared/errors/http-errors.ts`; `src/shared/logger/index.ts`; `src/middlewares/error.middleware.ts`; `src/middlewares/validate.middleware.ts`; `src/middlewares/auth.middleware.ts`; `src/modules/users/user.routes.ts`; `src/modules/auth/auth.routes.ts`; `src/modules/auth/auth.schemas.ts`; `src/modules/auth/auth.service.ts`; `src/modules/users/user.service.ts`; `prisma/schema.prisma`.
- ADRs relacionados: [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md).
