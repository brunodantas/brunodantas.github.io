---
title: "Personal AI Alignment"
description: "You can't retrain the model, but you own the harness around it: standing instructions, memory, skills, and guardrails that align AI to how you work."
date: 2026-07-27
categories: [ai, llm, workflow]
tags: [ai-alignment, llm, context-engineering, agent-harness, claude-code, productivity]
author: brunodantas
image: /assets/images/og/personal-ai-alignment.png
seo:
  type: BlogPosting
---

Anyone who has used any of the AI assistants enough eventually notices that their actions sometimes just don't match with our will. We think that AI just isn't there yet. It can feel really dumb sometimes.

Talking to people about their use cases, their problems include:
- Correcting the AI often.
- Having to explain the same steps for some procedure countless times.
- The AI assistant forgets things and/or doesn't perform as expected after a while.

This gap between what you want and what the assistant actually does is called the *AI Alignment problem* in the literature. In practice, it basically means a waste of time and resources.

In this post, we'll see some approaches that can help us move towards a smaller alignment gap in general use cases, rather than specific domains, leading to better usability and less frustration.

This is aimed at most kinds of AI users, rather than only technical people.

> For some context: in the past year, as I explored more deeply the current capabilities of AI assistants, I noticed that I started getting less and less frustrated with AI behavior. That motivated the writing of this post, since I believe a lot of the knowledge of modern practices in the field hasn't reached mainstream yet.
> 
> Also note: I'm biased towards software engineering concepts. However, I believe that the concepts in this post can be used in different contexts as well, as I have been using with success.

## Framing

Let's start with some framing and demystification in plain terms. Refer to the [Hugging Face glossary](https://huggingface.co/blog/agent-glossary) if you'd rather see technical definitions.

In the beginning there was the *prompt*. That has been the main way to interact with AI. The clearer the prompt, the better the results, generally.

Then, *context management* came into play by attaching documents to prompts, as well as managing separate sessions (a.k.a chats).

The third, more recent keyword to have in mind is the *[harness](https://huggingface.co/blog/agent-glossary#harness)*. It contains everything that combines with the actual model to produce results. This post covers the components of the harness and how we can have them work in our favor.

> Q: Doesn't AI learn from us while we interact already?
> 
> Only vaguely. While ChatGPT and others keep some information from chat to chat, that is far from the power of maintaining a good harness, in terms of efficacy etc.

When we pair a harness with a model (e.g. GPT 5, Opus 5 etc), that's called an *agent*. That is basically what makes AI feel like it's yours. It has been becoming evident that improving a harness is the best way to improve results, because that's how one can steer the agent effectively. Though getting a better model also helps, if you can.

With this in mind, we'll go over the components of the harness, how to set them up and what to aim for. Everything listed should be possible in regular free AI chat apps, though tools like Claude Code certainly help in maintaining these in persistent files in your computer.

## Baseline

As mentioned before, the standard use case of AI assistants generally involves interacting with a chat interface via prompting plus some vague memory mechanism that each session interacts with. Fortunately, we don't have to rely on vague memories anymore.

There are multiple levels of memory layers we can explore, but perhaps the most relevant is the [`AGENTS.md`](https://agents.md) standard. Simply put, it's about having **a prompt that's executed at the start of every session**. Some benefits are:

- You have full control of what it contains, rather than relying on volatile memories.
- You don't need to remind the AI of things you want it to always remember.
- Follow the DRY principle (Don't Repeat Yourself).

All you need to do is ask your AI how to set up an equivalent to `AGENTS.md` so you can use it, or set it up in Custom Instructions. Talk with it about what it should contain and it will help you create a good one. Note that, since this will be executed on every session: *simpler is better*.

Now, what can this get us in practice?

### Example

Let's go through some examples from my personal `CLAUDE.md` (Anthropic's variation of `AGENTS.md`).

First, **communication rules**.

> Be extremely concise: say it in as few words as possible.
>
> One idea per line. Blank line between chunks. Bullets over prose. No preamble, don't restate what I already know.

These are really important if you're someone who reads AI output all day and doesn't have great eyesight.

> Push back plainly when you see a flaw or a better path. Surface risks and sprawl early; don't soften them into hedges.

A rule against following directions blindly.

> No em dashes. Rewrite with commas, colons, parentheses, or two sentences. En dashes for numeric ranges like 3–5 are fine.

Personal preference. I never liked those dashes.


Then, rules on "**how we work**".

Note: these will get a bit technical because they're tailored to my profession.

> Spec-first for non-trivial work. If I didn't point you at a plan or spec, offer to draft one before building.

Rule against acting before planning. We'll dive deeper into this later.

> Ground before you build: open the real artifact (existing code, a sibling, the source doc) and match its conventions, structure, and domain terms.

Rule against mistakes related to drifting off of project standards.



Finally, **standards**.

> Architectural characteristics first. My defaults: Maintainability, Simplicity, Reliability.

Engineering standards consist of trade-offs. These are my personal choices.


## Memory

While this whole post is arguably about AI memory, now we're gonna look into some habits and improvements around that.

### Context

Starting with the **Context**: this is everything that a single chat session keeps in "active" memory. The more relevant context it has for its current task, the better the model generally is at addressing it. However, researchers have found that there is something called [Context Rot](https://www.trychroma.com/research/context-rot) (which people also call the Dumb Zone) where AI models start to perform worse when their context gets too big (at 150k tokens or so, according to Matt Pocock).

What does this mean for us?

For better efficacy, start new sessions often. If needed, ask for a handoff from the old session so that the new one can start fresh with just enough context to keep going. Another point for "simpler is better".

### Memory Management

Those memories that get saved by the model automatically can be tweaked.

Depending on your specific application, you can run the command `/consolidate-memory` from Anthropic's plugin (or an equivalent) to fix stale memories, prune irrelevant ones, etc.

Another interesting command is called `/insights` (or equivalent). On Claude, it shows a report on how you prompt it: what you do well and what could improve. Great for self reflection.

### Privacy

This is something most people don't seem to think about when using these tools: your information goes to servers in giant companies and stays there. It's generally better to be safe and to not share sensitive information with them. The rule of thumb is to think of it as "emailing a stranger". Would you tell anyone the things you've been telling them?

## Procedures

Perhaps the most advanced topic in this post, as not all AI applications seem to support it yet.

Procedures (a.k.a the [skill](https://agentskills.io) standard) denote the idea of storing how-tos so that the AI can repeat something we told it to do. It's a good approach because:

- It's more reliable than the builtin memory.
- You don't need to teach it again every time how to do something (DRY).
- You can use skills that other people have written, and there are great ones out there.

To demystify all this, skills are just prompts that you *invoke* with special commands. So if you don't have support for them, you can always copy/paste from somewhere.

For a half-serious example, I've shared my own [How Am I Driving](https://github.com/brunodantas/how-am-i-driving) skill, which produces some feedback around your AI usage.

But my favorite skills by far come from the package called [mattpocock/skills](https://github.com/mattpocock/skills). While focused on Software Engineering, I've used its `grill-me` skill uncountable times in many contexts, including in scaffolding and researching for this post. The entire skill is shown below.

> Interview me relentlessly about every aspect of this until we reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.
> 
> Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.
> 
> If a fact can be found by exploring the environment (filesystem, tools, etc.), look it up rather than asking me. The decisions, though, are mine — put each one to me and wait for my answer.
> 
> Do not act on it until I confirm we have reached a shared understanding.

As I'd say, understanding the problem is the most important part of problem solving.

Also, reusing existing skills is certainly simpler than rewriting them every time. And simpler is better.

## Guardrails

It's written all over: AI makes mistakes.

The logical conclusion is that it needs guardrails. When you don't have a harness, you must be extra careful, because then, you are the harness. If you have a written harness though, a good amount of the guardrails can be automated.

That generally means having a **never do X without asking me first** in your AGENTS.md (or equivalent).

## Compounding

Each tip we saw here produces assets, which you can carry and tweak in a loop of continuous improvement.

Note that there is evidence pointing to the [harness being highly significant](https://www.langchain.com/blog/improving-deep-agents-with-harness-engineering) to real world gains. This is where we can make a difference and reach our own personal alignment.


## See also

- [ChatGPT Custom Instructions](https://help.openai.com/en/articles/8096356-chatgpt-custom-instructions)
- [ChatGPT Memory FAQ](https://help.openai.com/en/articles/8590148-memory-faq)
- [Claude memory and chat search](https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context)
- [Import and export your memory from Claude](https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude)
- [Skills in ChatGPT](https://help.openai.com/en/articles/20001066-skills-in-chatgpt)
