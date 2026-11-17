# Da Reunião ao Documento: Design Docs Gerados por IA

> **Status:** em andamento. Fases 0 e 1 concluídas; os documentos ainda não foram redigidos.

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

## Prompts customizados

Prompt da Fase 1, que gerou o inventário:

```text
Monte uma tabela única, em ordem cronológica, classificando cada ponto substantivo da reunião: decisão fechada, requisito funcional, requisito não funcional, restrição, alternativa descartada (com o motivo dito na reunião), item adiado para fase futura, questão em aberto, detalhe técnico secundário, informação de contexto ou prazo. Cada linha com timestamp, falante e citação curta. Destaque à parte: falas corrigidas depois, e tudo o que foi decidido após o resumo final. Eu confiro por amostragem e decido as ambiguidades que você apontar. Depois disso, grave o inventário final, já com as decisões tomadas, em docs/INVENTARIO.md: ele é o gabarito de todos os documentos e do Tracker.
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


## Como navegar a entrega

Os documentos ficam em `docs/` (`PRD.md`, `RFC.md`, `FDD.md`, `TRACKER.md` e `adrs/`). O inventário da transcrição está em `docs/INVENTARIO.md`. Os caminhos finais e a ordem de leitura sugerida entram aqui no fechamento.
