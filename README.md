# Da Reunião ao Documento: Design Docs Gerados por IA

> **Status:** em andamento. Fases 0 a 4 concluídas: o inventário, os 7 ADRs, o RFC e o FDD estão prontos; PRD e Tracker ainda não foram redigidos.

## Sobre o desafio

Este repositório contém a entrega do desafio "Da Reunião ao Documento", do MBA em IA da Full Cycle. A tarefa é transformar a transcrição de uma reunião técnica de cerca de 55 minutos (`TRANSCRICAO.md`) e o código de um Order Management System (Node.js, TypeScript, Express, Prisma e MySQL) no pacote de design docs da feature "Sistema de Webhooks de Notificação de Pedidos": PRD, RFC, FDD, ADRs e um Tracker de rastreabilidade.

O enunciado original está no [repositório base do desafio](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia).

## Ferramentas de IA utilizadas

**Claude Code** (extensão do VS Code): modelo Fable 5.2, com esforço configurado para Alto. É a única ferramenta de IA utilizada. 

A ferramenta foi utilizada para ler o código e a transcrição, mapear os pontos de integração, montar o inventário da reunião e conferir por comando cada citação e cada caminho de arquivo. 

A ferramenta também foi utilizada para gerar os prompts e os rascunhos dos documentos mencionados acima, mas não para revisar ou aprovar. 

A revisão e aprovação de cada fase e documento é feita por mim. Sou eu também quem definiu as regras do processo, decidi como registrar cada divergência e cada ambiguidade e aprovei cada etapa antes de seguir.

## Workflow adotado

Fixei três regras antes de começar:

- Nenhum documento é considerado pronto sem aprovação explícita.
- Cada documento passa por um ciclo: prompt dirigido, rascunho, autocrítica da IA contra as fontes, revisão humana, correção com antes e depois.
- Os commits são feitos manualmente, um por fase aprovada.

Fases concluídas:

1. **Fase 0, contextualização.** A IA leu o enunciado, a transcrição e o código e devolveu um mapa dos pontos em que a feature toca o sistema, além de 9 divergências entre o que a reunião afirma sobre o código e o que o código realmente tem. Decidi como registrar cada uma.
2. **Fase 1, inventário da transcrição.** O resultado é uma tabela cronológica com 107 pontos da reunião, cada um classificado e com minuto, falante e citação. À parte ficaram 5 falas corrigidas depois, 2 decisões tomadas após o resumo final e a lista do que foi decidido mas não entrou no resumo. Depois da conferência, o inventário final, já com a leitura adotada para cada ambiguidade, foi gravado em [docs/INVENTARIO.md](docs/INVENTARIO.md). As 124 citações do arquivo foram conferidas por comando contra a transcrição. Esse inventário é o gabarito dos documentos.
3. **Fase 2, ADRs.** As decisões, como quais ADRs deveriam ser criados e seus nomes, foram definidas antes de qualquer redação dos documentos propriamente ditos, como o enunciado sugere. Aprovei uma lista de 7 ADRs e o prompt antes da redação. A IA redigiu os rascunhos e entregou a autocrítica contra as fontes; as correções vieram em voltas sucessivas, cada uma com antes e depois dos trechos alterados. Antes de aprovar, pedi duas auditorias independentes: em cada uma, um agente de IA sem acesso à conversa confrontou os ADRs com a transcrição, o código e o inventário, e cada achado foi conferido nas fontes antes de chegar a mim para decisão. A cada volta, um comando conferiu citações, minutos e falantes, caminhos de código e referências. Nesta fase o inventário também ganhou o registro das 9 divergências entre a reunião e o código, e passou a ter 139 citações, todas reconferidas. Os ADRs estão em [docs/adrs/](docs/adrs/), com índice em [docs/adrs/README.md](docs/adrs/README.md).
4. **Fase 3, RFC.** O RFC consolida a proposta técnica em cima dos ADRs: o fluxo da solução aponta para o ADR de cada passo em vez de repetir a justificativa. Defini o autor e a data dos metadados; os revisores são os cinco participantes da reunião. O ciclo foi o mesmo dos ADRs: prompt dirigido, rascunho, autocrítica contra as fontes, correções com antes e depois e duas auditorias independentes, que pedi antes de aprovar. Nesta fase ficou traçada a fronteira dos riscos entre os documentos: o RFC traz os de arquitetura e de segurança, o PRD os de negócio e de prazo, e o FDD os de implementação. O RFC está em [docs/RFC.md](docs/RFC.md).
5. **Fase 4, FDD.** É o documento em que a reunião tem menos a dizer: ela não deu nome de campo, status HTTP nem métrica. Aprovei uma convenção de três origens, visíveis em cada afirmação: fala da reunião, com minuto e falante; arquivo de código existente; ou proposta do documento, marcada e numerada. As 27 propostas estão reunidas na última seção do FDD, cada uma com a fala ou o arquivo que a tornou necessária, e continuam sendo propostas, sujeitas à revisão do time. O que não dava para propor com segurança ficou como questão em aberto. O ciclo foi o mesmo: prompt dirigido, rascunho, autocrítica contra as fontes, correções com antes e depois e duas auditorias independentes. O FDD está em [docs/FDD.md](docs/FDD.md).

## Prompts customizados

Prompt da Fase 1, que gerou o inventário:

```text
Monte uma tabela única, em ordem cronológica, classificando cada ponto substantivo da reunião: decisão fechada, requisito funcional, requisito não funcional, restrição, alternativa descartada (com o motivo dito na reunião), item adiado para fase futura, questão em aberto, detalhe técnico secundário, informação de contexto ou prazo. Cada linha com timestamp, falante e citação curta. Destaque à parte: falas corrigidas depois, e tudo o que foi decidido após o resumo final. Eu confiro por amostragem e decido as ambiguidades que você apontar. Depois disso, grave o inventário final, já com as decisões tomadas, em docs/INVENTARIO.md: ele é o gabarito de todos os documentos e do Tracker.
```

Prompt da Fase 2, usado para cada um dos 7 ADRs:

```text
Escreva o ADR "<título>" em docs/adrs/ADR-NNN-<titulo-em-kebab-case>.md, em português do Brasil, no formato MADR.

Fontes permitidas: somente TRANSCRICAO.md e o código do repositório. Use as linhas do inventário <I-xx a I-yy>.

Seções, nesta ordem: Status; Contexto; Decisão; Alternativas Consideradas; Consequências (Positivas, Negativas, Trade-off); Referências.

O que entra:
- Status: "Aceito", com a referência à reunião técnica e os participantes que decidiram o ponto.
- Contexto: o problema específico deste sistema que forçou a decisão, com o trecho de código afetado quando houver (caminho real).
- Decisão: uma única decisão, em uma a três frases afirmativas, com os números exatos ditos na reunião.
- Alternativas: só as que foram ditas na reunião, cada uma com o motivo do descarte, o minuto e o falante. Se a reunião não discutiu alternativa, apresente uma plausível e marque "não discutida na reunião".
- Consequências: positivas e negativas que a reunião reconheceu, com minuto e falante. Consequência derivada do código vem com o caminho do arquivo. O trade-off em uma frase no formato "aceitamos X para obter Y".
- Referências: minutos e falantes usados, arquivos de código citados e links relativos para os ADRs relacionados.

O que não entra:
- Detalhe de implementação que pertence ao FDD: schema de tabela, contrato HTTP, pseudocódigo, nome de campo que a reunião não disse.
- Motivação de negócio além de uma frase de contexto; o resto pertence ao PRD.
- Uma segunda decisão: se o texto precisar dela, aponte para o ADR correspondente.
- Benefício, risco ou número sem origem. Menção a IA.

Regras:
- Vale a versão final das falas corrigidas.
- Item adiado ou em aberto aparece só como consequência ou limitação, nunca como parte da decisão.
- Cada caminho de arquivo citado precisa existir no repositório.
- Cada citação precisa estar na fala daquele participante naquele minuto.

Tamanho: uma página, no máximo uma e meia.
```

Prompt da Fase 3, que gerou o RFC:

```text
Escreva docs/RFC.md, a proposta técnica do Sistema de Webhooks de Notificação de Pedidos, em português do Brasil, como documento submetido à equipe para revisão.

Fontes permitidas: TRANSCRICAO.md, o código do repositório e os 7 ADRs já aprovados em docs/adrs/. Use o inventário (docs/INVENTARIO.md) como gabarito, com as leituras AMB e as divergências DIV já adotadas.

Seções, nesta ordem: Metadados (autor, status, data, revisores); Resumo executivo; Contexto e problema; Proposta técnica; Alternativas consideradas; Questões em aberto; Impacto e riscos; Decisões relacionadas.

O que entra:
- Metadados: autor Rafael D'Addio, data de hoje, status "Em revisão", revisores = os cinco participantes da reunião com o papel de cada um.
- Proposta técnica: a solução em nível de arquitetura, do cadastro do webhook à DLQ, em um fluxo de poucos passos. Cada passo aponta para o ADR que o decide, com link relativo, em vez de repetir a justificativa.
- Alternativas consideradas: as de alcance arquitetural que a reunião descartou, cada uma com o trade-off que levou ao descarte, minuto e falante. As alternativas locais ficam nos ADRs; cite-as em uma linha.
- Questões em aberto: só o que a reunião adiou ou não decidiu, separando "não decidido" de "adiado para fase futura". Questão que nasce do código entra com o caminho do arquivo.
- Impacto e riscos: o que muda no sistema existente (com caminho real) e os riscos que a reunião reconheceu ou que o código mostra, com o tratamento dado na reunião quando houve.

O que não entra:
- Detalhe do FDD: contrato HTTP, schema de tabela, matriz de erros, pseudocódigo, lista de headers campo a campo.
- Requisitos funcionais um a um, métricas e público-alvo: pertencem ao PRD.
- Justificativa completa de cada decisão: pertence ao ADR.
- Qualquer afirmação sem origem na transcrição ou no código: nada de consequência, benefício ou vínculo causal deduzido. Mitigação que a reunião não deu não é inventada. Menção a IA.

Regras:
- Entre duas falas em conflito, vale a mais nova.
- Item adiado ou em aberto nunca aparece como parte da proposta.
- Rota citada de fala fica como foi dita; rota citada do código usa o caminho real.
- Cada caminho de arquivo precisa existir; cada referência [hh:mm] Nome precisa corresponder ao que a pessoa disse naquele minuto.

Tamanho: 2 a 4 páginas.
```

Prompt da Fase 4, que gerou o FDD:

```text
Escreva docs/FDD.md, o detalhamento de implementação do Sistema de Webhooks de Notificação de Pedidos, em português do Brasil, acionável o bastante para um desenvolvedor começar a codar.

Fontes permitidas: TRANSCRICAO.md, o código do repositório, os 7 ADRs e o RFC já aprovados. Use o inventário (docs/INVENTARIO.md) como gabarito, com as leituras AMB e as divergências DIV adotadas.

Seções: Contexto e motivação técnica; Objetivos técnicos; Escopo e exclusões; Modelo de dados; Fluxos detalhados (criação do evento na outbox, processamento pelo worker, retry, DLQ); Contratos públicos (endpoints com exemplo de request e response, headers, status codes); Matriz de erros com códigos WEBHOOK_*; Estratégias de resiliência; Observabilidade (métricas, logs, tracing); Integração com o sistema existente; Dependências e compatibilidade; Critérios de aceite técnicos; Riscos e mitigação.

Origem de cada afirmação, sempre visível:
- Reunião: minuto e falante.
- Código: caminho de arquivo que existe. Quando a reunião não definiu um ponto e o código tem um padrão, siga o padrão e cite o arquivo.
- Proposta: ponto que nem a reunião nem o código definem e sem o qual o documento não é implementável. Marque como "Proposta P-nn", diga a fala ou o arquivo que tornou a definição necessária e reúna todas em uma seção final. Não proponha número sem origem.
- O que não der para propor com segurança vai para "Questões em aberto".

O que entra:
- Contratos com o prefixo /api/v1 do código, campos em camelCase nos endpoints e snake_case no payload do webhook, como dito na reunião.
- "Integração com o sistema existente": o que muda ou como é usado cada arquivo real, apontando para o ADR-006 em vez de repetir a tabela dele. Arquivos novos aparecem só pelo nome, com o diretório existente à parte.
- Riscos de implementação apenas; os de arquitetura e segurança estão no RFC, os de negócio e prazo no PRD.

O que não entra:
- Justificativa das decisões (ADRs), proposta e alternativas (RFC), requisitos de produto, métricas de sucesso e prazo (PRD).
- Pontos que a reunião não tratou e que o autor decidiu deixar fora de todos os documentos.
- Pergunta virando afirmação, oferta virando fato, verbo mais forte que a fala. Menção a IA.

Regras: entre falas em conflito vale a mais nova; item adiado ou descartado só aparece em exclusões; cada caminho citado precisa existir; cada referência precisa corresponder ao que a pessoa disse naquele minuto.
```

## Iterações e ajustes

### Fases 0 e 1

Nenhum documento foi redigido até aqui, então ainda não houve iteração de documento. Os ajustes destas fases foram sobre as fontes:

- **A reunião e o código não dizem a mesma coisa em alguns pontos.** As divergências foram detectadas pela IA na Fase 0. Três exemplos:
  - O replay exige role ADMIN, mas o cadastro de usuário é aberto e aceita essa role. Decidi registrar como risco, com fonte no código.
  - A reunião pede o código `WEBHOOK_INVALID_URL` e também diz que a validação de https fica no schema Zod, que hoje sempre devolve `VALIDATION_ERROR`. Decidi levar para questão em aberto.
  - O filtro de eventos aceita qualquer status, mas nenhuma transição leva a `PENDING`. Decidi registrar como limitação conhecida.
- **"5 tentativas" era ambíguo.** A ambiguidade foi apontada pela IA na Fase 1: pelos intervalos ditos na reunião, o número corresponde ao envio inicial mais 5 retentativas. Decidi adotar essa leitura e deixá-la explícita nos documentos.
- **"Os últimos 100 webhooks" podia ser regra ou exemplo.** A dúvida foi apontada pela IA na Fase 1, com três formas de registrar. Decidi tratar a fala como exemplo: o histórico de entregas segue a paginação que o projeto já usa, sem limite inventado.
- **Uma armadilha fora do plano inicial.** O plano previa três tipos de armadilha na transcrição: itens descartados ou adiados, falas corrigidas e decisões tomadas depois do resumo final. O inventário mostrou um quarto: itens decididos antes do resumo que ficaram fora dele, como o timeout de 10 segundos e o limite de 64 KB.

### Fase 2

Os 7 ADRs passaram por 8 voltas de revisão até a aprovação. Em cada volta, citações, minutos, falantes e caminhos de código foram conferidos por comando. Os ajustes principais:

- **A garantia do outbox estava maior do que a reunião decidiu.** O rascunho do ADR-001 dizia que toda mudança de status tem o seu evento registrado. Isso contradiz o filtro aplicado na inserção: sem webhook inscrito naquele status, nada é gravado. O erro foi apontado pela IA na autocrítica.
- **Consequências deduzidas, sem origem.** A autocrítica listou oito trechos que eram dedução lógica da IA, não ditos por ninguém na reunião, e propôs mantê-los marcados. Lembrei que o enunciado só aceita informação rastreável à transcrição ou ao código, pedi a conferência no enunciado e decidi que os trechos saíam. A regra passou a valer para os próximos documentos: nenhuma derivação própria, com a única exceção que o enunciado prevê, a alternativa plausível em ADR.
- **Conclusão tirada do código sem demonstração.** O ADR-006 dizia que a proteção do replay herdava a fragilidade do cadastro aberto de usuários. Decidi que, sendo conclusão tirada do código, ela devia estar documentada: o ADR agora mostra o caminho arquivo por arquivo.
- **Detalhes fora da altura do ADR.** Os índices da tabela, os itens do payload e o limite de 64 KB saíram dos ADRs e ficam reservados para o FDD. Foram apontados pela autocrítica e pela segunda auditoria.
- **Primeira auditoria independente.** Não achou fato errado, mas 12 inconsistências se confirmaram na conferência. Quatro exemplos: o ADR-006 listava Sofia entre quem decidiu um ponto fechado depois de ela sair da chamada; o ADR-003 citava a versão inicial de uma fala corrigida mais tarde; o ADR-002 dizia "exigiria" onde Diego disse "dá pra"; o trade-off do ADR-007 aceitava e obtinha a mesma coisa.
- **Correções que criaram problemas novos.** A segunda auditoria mostrou que três ajustes anteriores tinham piorado o texto. O encurtamento de um trecho do ADR-001 transformou uma pergunta de Bruno em afirmação. A retirada da revisão de segurança do ADR-004, somada à frase "a reunião não apontou custo", contradizia uma exigência de Sofia. O filtro do ADR-001, movido para as consequências, deixou a decisão incondicional; neste ponto os dois revisores discordaram entre si. Decidi devolver a revisão de segurança ao ADR-004 e manter o filtro como condição dentro da frase da decisão.
- **Erros no gabarito.** A segunda auditoria também achou dois erros no registro das divergências do inventário: uma fala de Sofia sobre roles sustentava algo que ela não disse, e "todas as rotas" ignorava a rota `/health`. Antes disso, ao gravar as divergências, a IA corrigiu uma contagem própria: as transições de status permitidas são 7, e não 8.

### Fase 3

O RFC passou por 6 voltas de revisão até a aprovação, com a mesma conferência por comando a cada volta. Os ajustes principais:

- **Frases mais fortes que as falas.** Foi o desvio mais frequente do rascunho. A autocrítica apontou quatro casos e as duas auditorias acharam outros do mesmo tipo. Perguntas viraram afirmações: "seria mais reativa", onde Bruno perguntou se a trigger não seria, e "ele recebe 50 chamadas", onde Diego perguntou. Uma oferta virou fato: "documentada no portal", onde Marcos disse "eu posso documentar". Um verbo cresceu: "pediu", onde Diego disse "vale registrar".
- **O filtro perdeu o cliente.** O RFC dizia que o evento é gravado quando "há webhook inscrito naquele status". Lido ao pé da letra, o webhook de qualquer cliente bastaria. A primeira auditoria apontou, e o texto passou a dizer "webhook do cliente do pedido".
- **Dois prazos para o mesmo cliente.** A IA registrou como ambiguidade as falas "fim do trimestre" e "fim de novembro". Decidi que, entre falas em conflito, vale sempre a mais nova, e que prazo e risco de negócio saem do RFC e ficam para o PRD.
- **Conteúdo na altura errada.** O código de erro para URL http, que na Fase 0 ficou como questão em aberto, estava entre as questões de arquitetura do RFC. A primeira auditoria lembrou que a própria Sofia disse não se tratar de decisão arquitetural; a questão foi para o FDD. O contexto, que repetia o ADR-001, foi resumido.
- **Uma correção que tirou demais.** Ao mover o prazo para o PRD, o RFC ficou sem a revisão de segurança antes do deploy, a única restrição que Sofia repetiu no fim da reunião. A primeira auditoria apontou, e a linha voltou, apontando para o ADR-004.
- **Um desvio que já estava nos ADRs.** A auditoria de fechamento notou que uma fala citada como origem era a pergunta de Bruno, e não a confirmação de Sofia. O mesmo padrão estava no ADR-004, já aprovado, e foi corrigido nos dois documentos.

### Fase 4

O FDD passou por 6 voltas de revisão até a aprovação, com a mesma conferência por comando a cada volta. Os ajustes principais:

- **A regra de não deduzir contra um documento que exige detalhe.** A regra fixada na Fase 2 não permitia um FDD implementável só com as falas. A IA propôs a convenção de três origens, e eu a aprovei. Aceitei as sete propostas de maior peso, como a granularidade da outbox e o significado do status "falhou", com a condição de que continuassem marcadas como propostas.
- **Afirmações sem marca.** Mesmo com a convenção, pontos de desenho passaram como se fossem fato. A autocrítica apontou nove e a primeira auditoria mais sete, entre eles os campos editáveis do webhook, a posição da chamada dentro da transação e o formato das respostas de rotação e de replay. Todos viraram proposta marcada.
- **Critério de aceite em risco.** A autocrítica notou que só um endpoint tinha request e response explícitos, e o enunciado pede quatro. Os sete endpoints passaram a ter os dois blocos.
- **Propostas que se contradiziam.** A primeira auditoria mostrou que verificar o limite de 64 KB na inserção faz a rota de mudança de status responder um erro novo, o que contradizia a proposta de que nenhuma rota existente muda. Mostrou também que o modelo não dizia se as tabelas novas têm chave estrangeira; a conferência no código confirmou que pedidos e clientes são apagados fisicamente, então a escolha muda o comportamento de rotas que já existem. Aceitei as recomendações: a exceção ficou declarada, e as chaves estrangeiras viraram uma proposta própria.
- **Um erro do módulo sem o prefixo.** O cadastro com cliente inexistente respondia `NOT_FOUND`, contra a fala de Larissa de que tudo no módulo leva o prefixo `WEBHOOK_`. A primeira auditoria apontou; o código passou a ser `WEBHOOK_CUSTOMER_NOT_FOUND`.
- **Erros sobre o código.** A primeira auditoria achou dois: o tipo `TxClient` não é exportado pelo arquivo de pedidos, e o worker não precisa criar uma instância do Prisma, porque o arquivo de configuração já a cria ao ser importado. A auditoria de fechamento achou um terceiro, na descrição do arquivo de rotas.
- **Os mesmos desvios de fala.** Duas perguntas de Bruno apareceram como afirmação ou como origem de uma decisão, e a secret vazada "em log de aplicação dele", do cliente, tinha virado risco de log da plataforma. As duas auditorias apontaram.
- **Uma correção que introduziu erro.** Ao aplicar a proposta das chaves estrangeiras, o texto passou a citar um campo que a tabela de tentativas não tem. A auditoria de fechamento apontou, e o texto foi corrigido.


## Como navegar a entrega

Os documentos ficam em `docs/` (`PRD.md`, `RFC.md`, `FDD.md`, `TRACKER.md` e `adrs/`). O inventário da transcrição está em `docs/INVENTARIO.md`. Os ADRs já estão prontos em `docs/adrs/`, com índice em `docs/adrs/README.md`, o RFC em `docs/RFC.md` e o FDD em `docs/FDD.md`. Os caminhos finais e a ordem de leitura sugerida entram aqui no fechamento.
