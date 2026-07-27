# Blog vocabulary

Canonical terms used across posts on this blog. Written to keep one concept to
one name, in English and in the Portuguese versions. Glossary only.

## Personal AI alignment

**Harness**:
Everything around a language model that a user controls: instructions, tools,
memory, guardrails, the loop. Everything except the weights. Canonical 2026
usage, shorthand *Agent = Model + Harness*.
_Avoid_: scaffolding, wrapper, setup

**Personal alignment**:
Aligning an assistant to one person's intent by shaping its harness, as opposed
to lab alignment, which changes the model itself through training.
_Avoid_: fine-tuning, training your AI, customization

**Baseline**:
The always-on half-lever: standing instructions that apply to every interaction.
Custom Instructions for consumers, `AGENTS.md` / `CLAUDE.md` for developers.
_Avoid_: instructions, system prompt, rules file

**Procedures**:
On-demand context: a named, reusable recipe for one job, invoked only when the
task calls for it. Saved prompts for consumers, Agent Skills for developers.
_Avoid_: workflows, prompts, templates, commands, **skills**

"Skills" is deliberately not the name of this lever. It is one vendor form of it,
and naming the pattern after its power form would make every consumer form read
as a degraded skill rather than a legitimate procedure. Agent Skills is still
named, prominently, inside the beat.

**Memory**:
Accreting context: durable facts that grow over time, in two stages, capture
then consolidate.
_Avoid_: knowledge base, notes, history

**Guardrails**:
The control half of the harness: limits on what the assistant may do, and the
checks that catch it when instructions are not enough.
_Avoid_: safety, permissions, constraints

**Context engineering**:
The umbrella discipline of choosing what goes into a model's context. Named once
in the post as the industry term for the three context levers.
_Avoid_: prompt engineering

**Context rot**:
Measured degradation in model accuracy as the input context grows, well before
the window is full. Industry term, backed by Chroma's 2025 benchmark.
_Avoid_: context bloat, drift

**Dumb zone**:
Matt Pocock's term for what context rot feels like from the user's side
(AI Coding Dictionary: dumb zone commonly begins ~125-150k tokens, debated).
Introduced after the canonical term, attributed to Pocock.

**Simpler is better**:
The through-value: the least context that does the job. Context is a budget, not
a bucket.
_Avoid_: less is more, minimalism

### pt-BR equivalents

Decided for the Portuguese version (2026-07-27). English terms stay in English,
glossed in Portuguese on first use: **harness** (glossed "arreio"), **skills**,
**baseline**, **Framing**, **Context Rot**, **Dumb Zone** (glossed "zona
burra"), **guardrails** (glossed "corrimãos"), **prompt**, **handoff**.
Translated: Motivação (Motivation), Memória (Memory), Contexto (Context),
Gestão de Memória (Memory Management), Privacidade (Privacy), Procedimentos
(Procedures), O Efeito Composto (Compounding), Veja também (See also). The
through-value renders as "quanto mais simples, melhor". Quoted artifacts
(CLAUDE.md excerpts, the grilling skill) stay verbatim in English, flagged as
such in the prose, with a short Portuguese gloss after each.
