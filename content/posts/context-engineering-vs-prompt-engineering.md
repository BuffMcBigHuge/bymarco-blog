---
title: "Prompt engineering is getting demoted. Context engineering is the real job now."
date: 2026-03-11T19:00:00-05:00
draft: false
description: "Prompt engineering still matters, but it’s no longer the main bottleneck. In modern AI systems, context engineering, the design of what the model sees, remembers, and can act on, is where the real leverage is."
summary: "Prompt engineering tunes a single interaction. Context engineering designs the information environment across interactions. If you're building serious AI systems, that's the difference that matters."
tags:
  - ai
  - prompt-engineering
  - context-engineering
  - agents
  - llm
  - ai-systems
categories:
  - ai
author: "Marco"
showToc: true
TocOpen: false
comments: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
cover:
  image: "/images/posts/context-engineering/001-editorial-hero-illustration-for-a-techni.png"
  alt: "Editorial illustration showing a model operating inside a rich context environment with memory, retrieval, tools, and state"
  caption: "Prompts matter. Context matters more."
  relative: false
  hidden: false
---

The phrase “prompt engineering” had a good run.

It captured a real skill: learning how to ask models for better outputs. But the center of gravity has moved. If you’re doing anything beyond one-off chatbot interactions, the hard part isn’t wording the prompt. It’s designing the **context** the model operates inside.

That’s why “context engineering” is showing up everywhere now.

Not because the industry needed another buzzword. Because the bottleneck changed.

## The short version

**Prompt engineering** is about crafting the instruction.

**Context engineering** is about designing the information environment around that instruction:

- what the model knows
- what it sees right now
- what it can retrieve
- what tools it can use
- what memory it carries forward
- what constraints and handoff artifacts shape its behavior

Prompting still matters. But in serious systems, prompt quality is rarely the main limiter anymore.

The bigger limiter is whether the model has the **right information, in the right form, at the right time**.

## Why prompt engineering felt so important

In the early phase of LLMs, most usage looked like this:

- one chat
- one user
- one task
- one output

In that setup, the wording of the prompt mattered a lot. Small phrasing changes could flip quality from useless to great. People learned tricks:

- assign a role
- give examples
- ask for step-by-step reasoning
- specify format
- constrain output

That was real. It still works.

But it’s basically **local optimization**. You are tuning one exchange.

That’s fine when the task is “write me a summary.” It breaks down when the task is “help me run a product, operate a codebase, research a market, or act as an agent over time.”

## Why context engineering is replacing it

Modern AI systems fail less often because of a bad sentence in the prompt, and more often because of:

- missing context
- stale context
- irrelevant context
- bad retrieval
- weak memory
- unclear task state
- too much junk in the window
- no reliable handoff between steps
- tools with no guardrails
- zero observability

That’s context engineering territory.

One X post put it cleanly:

{{< linkcard
  url="https://x.com/JoeJMarston/status/2031764471652196370"
  title="Joe Marston: prompt engineering → context engineering → workflow engineering"
  site="X"
  author="@JoeJMarston"
>}}

His framing is simple and useful:

- Level 1: prompt engineering
- Level 2: context engineering
- Level 3: workflow engineering

That sequence feels right. Prompting is how you talk to the model. Context engineering is how you make the model an insider. Workflow engineering is how you make it do useful work repeatedly.

## The real difference

Here’s the cleanest distinction I can make.

### Prompt engineering asks:

- How should I phrase this request?

### Context engineering asks:

- What should the model know before I ask?
- What files, memory, tools, schemas, examples, and constraints should be in scope?
- What should be retrieved vs persisted?
- What should be excluded to avoid confusion?
- How does state survive across turns, sessions, and agents?
- How do I make the next step start from something better than “here’s a giant blob of chat history”?

Prompt engineering is **message design**.

Context engineering is **system design for model cognition**.

That’s a much bigger job.

## The model is not the whole product

One of the better pushes against the hype came from Krishna Gade:

{{< linkcard
  url="https://x.com/krishnagade/status/2031777177193369624"
  title="Krishna Gade: the emerging problem is runtime infrastructure, not just prompt wording"
  site="X"
  author="@krishnagade"
>}}

His point is sharp: the industry keeps renaming the same family of problems, but the real pressure is moving into runtime infrastructure, execution environments, and guardrails for probabilistic systems.

I think that’s mostly right.

“Context engineering” is useful if it makes people think beyond clever prompting. It becomes useless if it turns into another shallow label for the same old prompt hacks.

The durable version of the idea is this: **model quality is downstream of context quality**.

## What context engineering actually includes

When people say “context engineering,” they usually mean some mix of the following:

### 1) Retrieval

What gets pulled into the window at inference time?

This includes:

- RAG pipelines
- search over docs / code / tickets / prior chats
- reranking
- chunking strategy
- metadata filters
- recency and trust weighting

Bad retrieval poisons everything. The model can be brilliant and still act dumb if the wrong stuff gets loaded.

### 2) Memory

What survives beyond the current turn?

This includes:

- user preferences
- project facts
- prior decisions
- durable constraints
- task state
- summaries / handoff docs
- episodic logs vs curated memory

Reddit is pretty clear on this one: people feel the pain of agent forgetting constantly.

{{< linkcard
  url="https://www.reddit.com/r/ChatGPTCoding/comments/1rps784/built_an_open_source_memory_server_so_my_coding_agents_stop_forgetting_everything_between_sessions/"
  title="Reddit: coding agents stop forgetting everything between sessions"
  site="Reddit"
  author="u/Shattered_Persona"
>}}

The top comments are revealing. People aren’t just asking for more memory. They’re asking for the right kind:

- working state vs long-term memory
- decay / relevance management
- explicit handoff files
- protection against compaction drift

That’s not prompt engineering. That’s information architecture.

### 3) Tool context

What can the model do, and what does it know about those tools?

A model with tool access but no useful context is chaos with a keyboard.

A good tool layer includes:

- tool descriptions that are actually discriminative
- clear input/output contracts
- permission boundaries
- observable actions
- recovery paths when a tool fails
- state written out after actions, not held implicitly in chat mush

### 4) State management

Where does “what we are doing” live?

This is the part most people skip.

Strong systems externalize state:

- task files
- scratchpads
- spec docs
- checklists
- structured outputs
- handoff artifacts between steps or agents

Weak systems hope the model remembers what happened 40 turns ago.

Reddit users keep rediscovering the same thing: mid-session state loss is brutal, and explicit state files often beat magical memory.

### 5) Context hygiene

More context is not always better.

This is probably the single most misunderstood point.

A lot of teams overload the model with everything they have:

- entire repos
- giant knowledge bases
- every prior conversation
- all user preferences
- 12 tools
- stale summaries
- irrelevant examples

Then they wonder why outputs get weird.

One of the best Reddit takes on Claude’s new memory feature made exactly this point: the problem isn’t memory, it’s retrieval.

{{< linkcard
  url="https://www.reddit.com/r/ClaudeAI/comments/1rhx7pq/new_anthropic_introduces_a_memory_feature_that/"
  title="Reddit: portable memory is useful, but naive memory dumping hurts relevance"
  site="Reddit"
  author="u/Zealousideal_Disk164"
>}}

That’s the whole game. Not “how do I give the model more?” but “how do I give it the right slice?”

## Why this matters more for agents than chatbots

In single-turn chat, prompt engineering can carry a lot.

In agentic systems, it can’t.

Because the system now has to:

- interpret goals
- gather information
- choose tools
- take actions
- recover from errors
- preserve state
- hand off work
- stay aligned over time

That’s where context quality dominates.

A small X post said it better than most long essays:

{{< linkcard
  url="https://x.com/alex98075wa/status/2031755753967775780"
  title="Autonomy without proper context is just scheduled chaos"
  site="X"
  author="@alex98075wa"
>}}

Exactly.

An autonomous system without context engineering isn’t autonomous. It’s a recurring mistake.

## The best mental model: prompt = query, context = operating environment

If I had to compress the whole distinction into one line:

**A prompt is a request. Context is the world the request lives inside.**

That “world” includes:

- the current task
- the user
- prior decisions
- domain docs
- accessible tools
- memory
- guardrails
- output schema
- evaluation criteria

Once you see it that way, the hierarchy becomes obvious.

You can improve a bad system a bit with a better prompt.

You can improve a good system a lot with better context design.

## The trap: renaming without leveling up

There’s also some justified skepticism here.

Plenty of people are rolling their eyes and saying the industry is just relabeling the same thing every quarter:

- prompt engineering
- context engineering
- harness engineering
- intent engineering
- whatever comes next

They’re not fully wrong.

But there *is* a real shift underneath the rebrand. The useful distinction is not whether the names are perfect. It’s whether the work changed.

And the work did change.

As models got better, the prompt stopped being the main source of leverage. The leverage moved outward into:

- data access
- memory
- retrieval
- orchestration
- runtime control
- evaluation
- observability

That’s a more serious stack.

## What people on X and Reddit are converging on

After looking through recent X posts and Reddit threads, the recurring pattern is pretty consistent:

### 1) Prompting still matters, but mostly as a thin interface layer

You still need clear instructions. You still need good task framing. But this is now the easy part.

### 2) Memory is becoming a first-class primitive

People are tired of agents forgetting preferences, task state, codebase decisions, or what happened before compaction. This is one of the clearest pressure points in Reddit discussions.

### 3) Good context is selective, not maximal

The best setups don’t dump everything into the model. They filter, summarize, retrieve, and externalize state.

### 4) Real-world agent quality depends on boring systems work

Not prompt poetry. Stuff like:

- schemas
- state files
- retrieval tuning
- evals
- permissions
- tool contracts
- logs
- fallback behavior

### 5) The next step after context engineering is workflow engineering

Once context is solid, the problem becomes sequencing and governance: who does what, in what order, with what checks.

That’s why the “context engineering” conversation keeps bleeding into agent infrastructure.

## My take

Prompt engineering isn’t fake. It’s just shrinking into the role it probably should’ve had all along.

It’s a useful skill. It is not the main discipline.

If you’re building anything serious, the bigger question is no longer:

> What’s the perfect prompt?

It’s:

> What information environment makes good behavior likely?

That includes what the model sees, what it remembers, what it can touch, what state it leaves behind, and how the system catches mistakes before they cascade.

That’s context engineering.

And honestly, even that may end up being a transitional label. Because once you push far enough, you realize the real work is broader still:

- context
- workflows
- evals
- governance
- runtime safety
- system design

The prompt was never the whole product. It was the visible tip.

## Practical advice

If you’re still spending most of your energy tweaking prompts, I’d shift your attention to these 5 questions:

### 1) What should be persistent vs ephemeral?

Separate durable memory from task-local scratch space.

### 2) What should be retrieved vs preloaded?

Don’t stuff everything into context by default.

### 3) Where does task state live outside the chat?

Use files, specs, summaries, or structured state, not just conversation history.

### 4) What tools does the model actually need?

Most agent systems are over-tooled and under-governed.

### 5) How do you know when the system is drifting?

Add evals, logs, checkpoints, and handoff artifacts.

That’s where the real performance gains are now.

## Bottom line

Prompt engineering helps you ask better.

Context engineering helps the system think with the right materials.

That’s the difference.

And as AI moves from chatbot novelty to actual operational layer, the second one matters a lot more.
