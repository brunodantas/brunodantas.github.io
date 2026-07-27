---
title: "Alinhamento da IA Pessoal"
description: "Por que assistentes de IA vivem errando o que você quer, e a solução que todos podem aplicar: um harness (arreio) de instruções, memória, skills, e guarda-corpos que torna o agente seu."
date: 2026-07-27
lang: pt_BR
categories: [ai, llm, workflow]
tags: [ai-alignment, llm, context-engineering, agent-harness, claude-code, productivity]
author: brunodantas
image: /assets/images/og/alinhamento-da-ia-pessoal.png
seo:
  type: BlogPosting
---

> Este post também está disponível [em inglês](/blog/personal-ai-alignment/).

Qualquer pessoa que já usou assistentes de IA o suficiente eventualmente percebe que as ações deles às vezes simplesmente não batem com a nossa vontade. A gente pensa que a IA ainda não chegou onde devia. Às vezes ela parece bem burra.

Conversando com as pessoas sobre seus casos de uso, os problemas incluem:
- Corrigir a IA com frequência.
- Ter que explicar os mesmos passos de algum procedimento incontáveis vezes.
- O assistente esquece as coisas e/ou deixa de agir como esperado depois de um tempo.

Essa distância entre o que você quer e o que o assistente realmente faz é chamada de *problema do Alinhamento de IA* na literatura. Na prática, ela basicamente significa desperdício de tempo e recursos.

Neste post, vamos ver algumas abordagens que podem nos ajudar a diminuir essa distância de alinhamento em casos de uso gerais, em vez de domínios específicos, levando a mais usabilidade e menos frustração.

Este post é voltado para a maioria dos tipos de usuários de IA, não só para o pessoal técnico.

## Motivação

Para contextualizar: no último ano, conforme explorei mais a fundo as capacidades atuais dos assistentes de IA, percebi que fui ficando cada vez menos frustrado com o comportamento da IA. Foi isso que motivou a escrita deste post, já que acredito que muito do conhecimento das práticas modernas da área ainda não chegou ao mainstream.

Uma observação: tenho um viés para conceitos de Engenharia de Software. Porém, acredito que os conceitos deste post podem ser usados em outros contextos também, como eu mesmo tenho conseguido com sucesso.

## Framing

Vamos começar enquadrando e desmistificando em termos simples. Consulte o [glossário do Hugging Face](https://huggingface.co/blog/agent-glossary) se preferir definições técnicas.

No começo era o *prompt*. Essa tem sido a principal forma de interagir com IA. Quanto mais claro o prompt, melhores os resultados, em geral.

Depois, veio a *gestão de contexto*: anexar documentos aos prompts e administrar sessões separadas (os chats).

A terceira palavra-chave, mais recente, é o *[harness](https://huggingface.co/blog/agent-glossary#harness)* ("arreio", o que te ajuda a guiar um cavalo). Ele contém tudo aquilo que se combina com o modelo em si para produzir resultados. Este post é sobre os componentes do harness e como fazê-los trabalhar a nosso favor.

> P: A IA já não aprende com a gente enquanto interagimos?
> 
> Só vagamente. O ChatGPT e outros até guardam alguma informação de um chat para outro, mas isso está longe do poder de manter um bom harness, em termos de eficácia etc.

Quando juntamos um harness a um modelo (GPT 5, Opus 5 etc.), isso se chama *agente*. É basicamente o que faz a IA parecer sua. Vem ficando evidente que melhorar o harness é a melhor forma de melhorar os resultados, porque é assim que se consegue guiar o agente de verdade. Mas note que usar um modelo melhor também ajuda, se você puder.

Com isso em mente, vamos passar pelos componentes do harness, como configurá-los e o que buscar. Tudo que está listado deve ser possível nos apps de chat de IA gratuitos, embora ferramentas como o Claude Code certamente ajudem a manter tudo isso em arquivos persistentes no seu computador.

## Baseline

Como mencionado antes, o caso de uso padrão dos assistentes de IA geralmente envolve interagir com uma interface de chat via prompts, mais algum mecanismo vago de memória com que cada sessão interage. Felizmente, não precisamos mais depender de memórias vagas.

Há vários níveis de camadas de memória para explorar, mas talvez o mais relevante seja o padrão [`AGENTS.md`](https://agents.md). Em resumo, trata-se de ter **um prompt que é executado no início de toda sessão**. Alguns benefícios:

- Você tem controle total do que ele contém, em vez de depender de memórias voláteis.
- Você não precisa relembrar a IA das coisas que quer que ela sempre lembre.
- Segue o princípio DRY (Don't Repeat Yourself/Não se Repita).

Basta pedir à sua IA para configurar um equivalente ao `AGENTS.md`, ou configurá-lo nas Custom Instructions (Instruções Customizadas). Converse com ela sobre o que ele deve conter e ela vai te ajudar a criar um bom arquivo. Note que, como isso será executado em toda sessão: *quanto mais simples, melhor*.

Agora, o que ganhamos com isso na prática?

### Exemplo

Vamos ver alguns exemplos do meu `CLAUDE.md` pessoal (a variação da Anthropic do `AGENTS.md`). As citações estão em inglês, como no meu arquivo original.

Primeiro, **regras de comunicação**.

> Be extremely concise: say it in as few words as possible.
>
> One idea per line. Blank line between chunks. Bullets over prose. No preamble, don't restate what I already know.

"Seja extremamente conciso..." "Uma ideia por linha." "Pontos em vez de prosa."

Essas são muito importantes se você passa o dia lendo saída de IA e não enxerga muito bem.

> Push back plainly when you see a flaw or a better path. Surface risks and sprawl early; don't soften them into hedges.

"Discorde quando vir uma falha ou alternativa melhor. Mostre os riscos cedo."

Uma regra contra seguir instruções cegamente.

> No em dashes. Rewrite with commas, colons, parentheses, or two sentences. En dashes for numeric ranges like 3–5 are fine.

"Nada de travessões..."

Preferência pessoal. Nunca gostei deles.


Depois, regras de "**como trabalhamos**".

Obs.: estas são mais técnicas porque são voltadas para a minha profissão.

> Spec-first for non-trivial work. If I didn't point you at a plan or spec, offer to draft one before building.

Regra contra agir antes de planejar. Vamos nos aprofundar nisso depois.

> Ground before you build: open the real artifact (existing code, a sibling, the source doc) and match its conventions, structure, and domain terms.

Regra contra erros relacionados a se desviar dos padrões do projeto.



Por fim, **padrões**.

> Architectural characteristics first. My defaults: Maintainability, Simplicity, Reliability.

Padrões de engenharia consistem em trade-offs. Estas são as minhas escolhas pessoais.


## Memória

Embora este post inteiro seja, de certo modo, sobre memória de IA, agora vamos olhar para alguns hábitos e melhorias em volta disso.

### Contexto

Começando pelo **contexto**: é tudo aquilo que uma sessão de chat mantém na memória "ativa". Quanto mais contexto relevante para a tarefa atual, melhor o modelo costuma se sair. Porém, pesquisadores descobriram que existe algo chamado [Context Rot](https://www.trychroma.com/research/context-rot) (que alguns chamam também de Dumb Zone, a "zona burra") em que os modelos de IA ficam piores quando o contexto fica grande demais (em torno de 150 mil tokens, segundo Matt Pocock).

O que isso significa para nós?

Para mais eficácia, comece sessões novas com frequência. Se precisar, peça um "handoff" da sessão antiga, para que a nova comece do zero com contexto suficiente para continuar. Mais um ponto para o "quanto mais simples, melhor".

### Gestão de Memória

Aquelas memórias que o modelo salva automaticamente podem ser ajustadas.

Dependendo da sua aplicação, você pode rodar o comando `/consolidate-memory` do plugin da Anthropic (ou um equivalente) para corrigir memórias desatualizadas, remover as irrelevantes etc.

Outro comando interessante é o `/insights` (ou equivalente). No Claude, ele mostra um relatório de como você conversa com ele: o que você faz bem e o que pode melhorar. Ótimo para auto-reflexão.

### Privacidade

É algo em que a maioria das pessoas parece não pensar ao usar essas ferramentas: as suas informações vão para servidores de empresas gigantes e ficam lá. Em geral, é melhor prevenir e não compartilhar informações sensíveis com elas. A regra geral é pensar como se fosse "mandar um e-mail para um estranho". Você contaria para qualquer um as coisas que anda contando para elas?

## Procedimentos

Talvez o tópico mais avançado deste post, já que nem todas as aplicações de IA parecem suportá-lo ainda.

Procedimentos (ou o padrão de [skills](https://agentskills.io)) denotam a ideia de armazenar instruções de "como fazer" para que a IA consiga repetir algo que ensinamos a ela. É uma boa abordagem porque:

- É mais confiável que a memória embutida.
- Você não precisa ensinar de novo toda vez como fazer algo (DRY).
- Você pode usar skills que outras pessoas escreveram, e existem umas ótimas por aí.

Desmistificando: skills são só prompts que você *invoca* com comandos especiais. Então, se você não tiver suporte a elas, sempre pode copiar e colar de algum lugar.

Como exemplo mais ou menos sério, eu publiquei a minha própria skill [How Am I Driving](https://github.com/brunodantas/how-am-i-driving) (Como Estou Dirigindo?), que produz um feedback sobre o seu uso de IA.

Mas as minhas skills favoritas, de longe, vêm do pacote chamado [mattpocock/skills](https://github.com/mattpocock/skills). Embora focado em engenharia de software, já usei a sua skill `grill-me` incontáveis vezes em muitos contextos, inclusive na estruturação e na pesquisa deste post. A skill inteira está abaixo, no original em inglês.

> Interview me relentlessly about every aspect of this until we reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.
> 
> Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.
> 
> If a fact can be found by exploring the environment (filesystem, tools, etc.), look it up rather than asking me. The decisions, though, are mine — put each one to me and wait for my answer.
> 
> Do not act on it until I confirm we have reached a shared understanding.

Como eu diria, entender o problema é a parte mais importante da resolução de problemas.

Além disso, reutilizar skills existentes certamente é mais simples do que reescrever tudo toda vez. E quanto mais simples, melhor.

## Guardrails

Está escrito em todo lugar: IA comete erros.

A conclusão lógica é que ela precisa de guardrails (guarda-corpos). Quando você não tem um harness, precisa de cuidado redobrado, porque, nesse caso, o harness é você. Mas se você tem um harness escrito, uma boa parte dos guardrails pode ser automatizada.

Isso geralmente significa ter um **nunca faça X sem me perguntar antes** no seu AGENTS.md (ou equivalente).

## O Efeito Composto

Cada dica que vimos aqui produz ativos, que você pode carregar e ajustar num ciclo de melhoria contínua.

Note que há evidências apontando que o [harness é altamente significativo](https://www.langchain.com/blog/improving-deep-agents-with-harness-engineering) para os ganhos no mundo real. É aí que podemos fazer diferença e alcançar o nosso próprio alinhamento pessoal.


## Veja também

- [ChatGPT Custom Instructions](https://help.openai.com/en/articles/8096356-chatgpt-custom-instructions)
- [ChatGPT Memory FAQ](https://help.openai.com/en/articles/8590148-memory-faq)
- [Memória e busca de chats no Claude](https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context)
- [Importar e exportar sua memória do Claude](https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude)
- [Skills no ChatGPT](https://help.openai.com/en/articles/20001066-skills-in-chatgpt)
