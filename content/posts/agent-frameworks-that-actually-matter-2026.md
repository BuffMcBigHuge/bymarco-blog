---
title: "The Agentic World, Updated: What’s Actually Better Than OpenClaw Now?"
date: 2026-03-16T11:45:00-04:00
draft: false
slug: "agentic-world-update-2026-better-than-openclaw"
url: "/posts/agentic-world-update-2026-better-than-openclaw/"
description: "An update to my earlier open-source agent-world survey: what LangChain Deep Agents, Hermes Agent, OpenViking, and the newest viral harnesses are improving, especially where they are genuinely better than OpenClaw."
summary: "This is a follow-up to my earlier agent-framework field guide. Since then, the center of gravity has shifted toward harnesses, context systems, and persistent runtimes. Deep Agents sharpens long-task execution, Hermes pushes the self-improving personal-agent model, and OpenViking attacks one of OpenClaw’s real bottlenecks: context architecture."
tags:
  - ai
  - agents
  - agentic-ai
  - langchain
  - hermes
  - openviking
  - tooling
  - developer-tools
categories:
  - ai-tools
author: "Marco"
showToc: true
TocOpen: false
hidemeta: false
comments: true
disableHLJS: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
  image: "/images/posts/agentic-world-update-2026/hero.png"
  alt: "Editorial illustration of the 2026 agent stack with context systems, memory layers, and autonomous workflows"
  caption: "The new agent world is less about prompt tricks and more about context, memory, and runtime architecture."
  relative: false
  hidden: false
---

A few weeks ago I wrote [First Chat, Then Code, Now Claw](/posts/first-chat-then-code-now-claw/), a field guide to the first big wave of open-source agent frameworks.

That post was about making the category legible.

This one is an update.

Because the space moved again.

The agent world is still messy, but it’s getting less fake.

A year ago, most “agent frameworks” were thin wrappers around tool calling with some grand language bolted on top. Now the category is starting to split into real layers with real tradeoffs.

That’s progress.

Over the last little while I’ve been reading through newer projects, docs, repos, and X chatter again, and a few names keep surfacing: **LangChain Deep Agents**, **Nous Hermes Agent**, **OpenViking**, plus a broader cluster of agent harnesses and frameworks that have been going semi-feral on GitHub and X over the last few weeks.

What’s interesting is that these tools are not all solving the same problem.

That’s where a lot of the confusion comes from. People compare them as if they’re direct substitutes. They usually aren’t.

## The short version

Here’s my read on the update:

- **OpenClaw** is still one of the most complete agent control planes, especially if you care about channels, nodes, scheduling, and broad day-to-day utility.
- But newer systems are getting **better in narrower ways**.
- **Deep Agents** looks better than OpenClaw at packaging the **harness pattern** for long-running, multi-step work.
- **Hermes Agent** looks better than OpenClaw at making the **persistent personal-agent + skill-learning** thesis feel like the center of the product.
- **OpenViking** looks better than OpenClaw at treating **context architecture** as a first-class infrastructure problem instead of a supporting feature.
- The broader trend is that the market is separating into: **runtime**, **harness**, **memory/context layer**, and **surface/product**.

That separation matters because the engineering problems are different.

## What changed since the last post

The old framing was: can the model call tools in a loop?

That’s no longer the hard part.

The new framing is closer to this:

- Can the agent stay coherent over long tasks?
- Can it manage context without drowning in its own history?
- Can it externalize state instead of pretending the chat log is a database?
- Can it operate safely across real surfaces like terminal, messaging, browser, and files?
- Can it improve or specialize over time without turning into entropy in a trench coat?

That’s why the newer frameworks are converging on the same building blocks:

- planning / todo systems
- file-backed or structured context
- subagents for isolation
- persistent memory
- tool contracts
- sandboxing
- scheduling
- observability

The details differ, but the architecture is getting more recognizable.

## 1) LangChain Deep Agents: the “agent harness” pattern gets productized

Since you meant **Deep Agents by LangChain specifically**, this is the one I’d put at the center of the post.

LangChain’s **Deep Agents** is one of the cleaner examples of where the category is headed.

Their own wording is useful here: they call it an **agent harness**, not just a framework. That’s actually the right term.

The point of a harness is not to give you raw primitives. It’s to give you an opinionated working setup for agents that need to do longer, messier work.

### What Deep Agents actually is

Based on the docs and repo, Deep Agents bundles a few core capabilities out of the box:

- planning and task decomposition
- filesystem-backed context management
- subagent spawning for isolated work
- memory across conversations/threads
- optional shell/sandbox execution
- a CLI on top of the SDK

The key design choice is that it treats **context management as infrastructure**, not as an afterthought.

That part is real.

The docs explicitly position the virtual filesystem as a way for the agent to offload large context, work with variable-length outputs, and avoid just stuffing everything into the prompt window. That is exactly the right instinct.

### What it’s good for

Deep Agents makes sense if you want to build agents that:

- work on **complex, multi-step tasks**
- need **planning** and progress tracking
- benefit from **subagents** for compartmentalization
- need a practical way to **externalize context**
- live inside the LangChain/LangGraph ecosystem already

This is especially relevant for research agents, coding agents, and workflow-like systems where a single flat chat loop becomes mush.

### What feels substantive vs what feels packaged

The substantive part:

- built-in context management primitives
- clear story for planning + subagents
- durable execution via LangGraph
- explicit distinction between lower-level framework and higher-level harness

The packaged part:

- some of the novelty is packaging and defaults, not a brand new conceptual breakthrough

That’s not a criticism. Packaging matters. Good defaults matter a lot. But people should understand what the actual contribution is.

Deep Agents is important because it standardizes a pattern many people were already rediscovering manually.

### Why it’s suddenly everywhere

On X, the viral framing around Deep Agents is pretty consistent:

- “open-source Claude Code alternative”
- “agent = model + harness”
- “planning + memory + context isolation”
- “LangChain just productized long-running agents”

That framing is a little reductive, but it explains the speed of the buzz.

People aren’t reacting to a new prompt trick. They’re reacting to a recognizable package of capabilities that many builders already believe they need.

The interesting part of the X chatter was not just the hype, it was the **specificity** of the hype. A lot of posts were fixated on context isolation, file-backed state, and long-horizon task handling. That’s a healthier sign than generic “this changes everything” posting.

### What feels improved over OpenClaw

OpenClaw’s strength is breadth.

Deep Agents’ strength is focus.

If I compare them honestly, Deep Agents does look better in a few specific ways:

- **clearer harness identity** for long-running work
- **more explicit context-isolation story**
- **better packaging for builders** who want the agent loop, not the full messaging/control-plane universe
- a tighter fit for people who think in terms of research/coding/task execution first

Where OpenClaw still wins is product surface. It’s a fuller assistant environment.

Where Deep Agents may be better is that it feels less like an everything machine and more like a purpose-built skeleton for serious agent workflows.

### My take

Deep Agents looks like one of the more sensible entries in the space because it’s not pretending raw tool calling is enough. It’s baking in the stuff serious builders end up adding anyway.

If you’re already in LangChain/LangGraph land, this probably reduces a lot of wheel reinvention.

If you’re not, it’s still a useful reference architecture for what a modern harness should include.

## 2) Hermes Agent: the personal-agent operating layer gets more serious

**Hermes Agent** from Nous Research is aiming at a bigger surface area.

This is not just “an agent SDK.” It’s closer to a full personal-agent environment that wants to persist, communicate, schedule, remember, and improve.

In other words, it’s trying to become the thing people keep vaguely gesturing at when they say they want an AI that “lives with them.”

### What Hermes is trying to do

From the docs and repo, Hermes is built around a pretty aggressive product vision:

- one agent process that can live across **CLI + messaging channels**
- persistent **memory** across sessions
- a **skills system** that can be created and improved from experience
- built-in **cron/scheduled automations**
- **subagents** for parallel work
- multiple terminal backends including local, Docker, SSH, Daytona, Modal, and more
- support for MCP, voice, context files, personalities, and long-running operation

The docs lean hard on the phrase “self-improving,” which I’d treat with some caution because that phrase gets abused a lot.

But unlike a lot of marketing pages, there is at least a concrete substrate under it:

- skill creation
- memory persistence
- cross-session recall
- periodic nudges to save useful information
- user modeling

That’s more specific than the usual “our agent learns” hand-waving.

### What Hermes is actually good for

Hermes looks strongest for people who want:

- a **persistent personal assistant** rather than a one-shot coding tool
- multi-surface operation across **Telegram, Discord, Slack, WhatsApp, Signal, CLI**
- strong emphasis on **memory + skills + scheduled work**
- a system that can run **somewhere else**, not just on the laptop in front of you

That last part matters.

A lot of agent products quietly assume the machine is your local interactive workstation. Hermes is much more comfortable with the idea that the agent lives on a VPS, serverless backend, or remote machine and you just talk to it through messaging.

That’s a different operational model, and for many use cases it’s the right one.

### The real thesis behind Hermes

The thesis seems to be:

> a useful personal agent is not just a model with tools, it’s a long-running software organism with memory, habits, surfaces, and procedural reuse

That’s the right thesis.

Whether every implementation detail lands perfectly is a separate question, but the direction is smart.

### What to be careful about

A few cautions:

- “self-improving” agents can drift into self-generated cruft if the memory/skill loop isn’t curated well
- more surfaces means more operational complexity and a larger security perimeter
- anything that touches messaging, files, shell, and scheduling at once needs real boundaries, not vibes

In other words, Hermes looks powerful, but this is a category where capability and blast radius rise together.

### What feels improved over OpenClaw

Hermes is especially interesting because it seems to believe the core product is not just “an agent with tools,” but **an agent that accumulates procedural memory and gets more reusable over time**.

OpenClaw absolutely has memory, skills, scheduling, and persistence. But Hermes pushes that learning loop much harder as the headline.

So if I compare them directly:

- OpenClaw feels stronger as a **broad assistant operating layer**
- Hermes feels stronger as a **self-improving personal-agent thesis**
- OpenClaw feels more like a control plane
- Hermes feels more like a long-lived collaborator that is supposed to grow habits and reusable procedural knowledge

That doesn’t automatically make Hermes better overall.

But it does make Hermes better at articulating, and maybe productizing, one of the biggest promises in this whole category.

### My take

Hermes is one of the more interesting projects right now because it’s trying to solve the full-stack personal-agent problem instead of hiding inside an SDK abstraction.

That ambition is both the appeal and the risk.

If you care about persistent assistants, not just agent demos, it’s worth watching closely.

## 3) OpenViking: context is becoming its own infrastructure layer

**OpenViking** is the one that caught my eye because it’s attacking a different problem.

Most agent tools assume context is an implementation detail. OpenViking treats it like a missing infrastructure primitive.

Their framing is blunt: memory, resources, and skills are usually fragmented across code, vector stores, files, prompts, and ad hoc retrieval systems. They want to unify that into a **context database** organized through a **filesystem paradigm**.

That sounds abstract at first, but the core idea is simple enough.

### What OpenViking is trying to fix

OpenViking’s pitch is basically this:

- traditional flat RAG is bad at global structure
- context gets fragmented across too many storage patterns
- long-running agents generate context continuously
- retrieval chains are hard to inspect and debug
- agent memory should include more than just past chat messages

That diagnosis is pretty solid.

The proposed solution is to organize context in a hierarchical, file-like way with:

- unified handling of **memory, resources, and skills**
- **tiered loading** to reduce token usage
- directory-style recursive retrieval plus semantic search
- visualized retrieval trajectories
- automatic session management and long-term extraction

### Why this matters

I think OpenViking matters because it points at a likely next step for the whole ecosystem:

**context stops being a bag of chunks and becomes an addressable operating environment**.

That’s a big deal.

Right now, a lot of “memory systems” are basically:

- embed everything
- query vaguely
- rerank
- hope the right fragments show up

That works up to a point. Then the agent becomes a confused intern with access to a warehouse.

A hierarchical context model could be much better for:

- project-based retrieval
- skills and resources that belong together
- selective loading
- observability when retrieval goes wrong

### What feels real vs aspirational

The real part:

- the problem is real
- the filesystem mental model is genuinely useful
- tiered context loading is the kind of thing agents need
- observability for retrieval paths is overdue

The aspirational part:

- context databases are still early, and the quality of the abstraction matters more than the ambition statement
- a lot depends on whether the hierarchy remains intuitive as real-world use gets messy
- if config and ops overhead get too heavy, many developers will bounce even if the theory is good

### What feels improved over OpenClaw

This is the sharpest comparison in the whole post.

OpenClaw already has memory and retrieval. But OpenViking is attacking the deeper problem underneath them: **how context itself is organized, addressed, loaded, and debugged**.

That matters because one of the real weak spots in almost every agent stack, not just OpenClaw, is that context becomes an opaque soup:

- some in files
- some in memory stores
- some in prompt scaffolding
- some in ad hoc retrieval
- some in tool output history

OpenViking’s filesystem-style context model looks like a genuine attempt to clean that up.

If it works well in practice, then yes, this could be a real improvement over OpenClaw in one important dimension:

**context architecture**.

Not assistant breadth. Not channels. Not overall product completeness.

But context architecture is a huge lever, and OpenClaw could plausibly benefit from ideas like this whether or not OpenViking itself becomes the winner.

### My take

OpenViking is interesting because it’s not trying to win by being “the agent.” It’s trying to become part of the substrate underneath agents.

That might end up being the bigger opportunity.

A lot of the next wave of agent quality gains will not come from smarter prompting. They’ll come from better context systems.

## 4) Other viral tools and harnesses worth tracking

Deep Agents, Hermes, and OpenViking are the three I’d highlight most from this pass, but they’re not alone.

A bunch of adjacent projects keep showing up in GitHub trending screenshots and X threads. I wouldn’t put all of them in the same bucket, but they’re clearly part of the same moment.

### OpenCode

OpenCode is still one of the biggest reference points in the harness conversation, especially on X where people keep grouping it with Deep Agents, Goose, and other coding-agent environments.

What matters about OpenCode is not just scale or stars. It helped normalize the idea that the model itself is not the product. The real differentiators are:

- context handling
- workflow ergonomics
- extensibility
- multi-model portability
- how well the tool survives long, messy sessions

It’s less “framework” in the abstract and more “proof that the harness layer is now a category.”

### Goose, ForgeCode, and similar coding-agent harnesses

These keep coming up in the same breath as Deep Agents for a reason.

They all sit in the same design neighborhood:

- terminal-first or dev-first agent workflows
- multi-step task execution
- file/tool access
- some notion of memory or reusable instructions
- stronger operational posture than a plain chatbot wrapper

I wouldn’t flatten them into one thing, but the market is obviously rewarding tools that feel like **work environments**, not just chat interfaces.

### DeerFlow and the ByteDance wave

DeerFlow keeps getting mentioned alongside OpenViking in “what’s viral on GitHub” posts.

That pairing is interesting.

It suggests ByteDance is not just shipping one-off demos, it’s shipping pieces of a larger belief about agent systems: memory, context, sandboxing, tools, long-running execution. Even when the individual projects differ, the direction feels coherent.

### Agency / company-of-agents style frameworks

There’s also a separate viral lane built around “an entire startup run by agents” style repos.

These are useful as signals, but I’d be more skeptical here.

They’re great at attention. Less clear that they’re solving the hard infrastructure problems as well as the better harness projects are.

If I’m choosing where to pay attention, I care more about:

- state
- isolation
- memory quality
- tool boundaries
- observability

…than whether the repo roleplays a 12-person company.

## 5) The broader wave: the stack is separating into layers

Once you look at these projects side by side, the category starts making more sense.

They’re not all fighting for the same box on the diagram.

### Layer A: agent runtimes / personal agent systems

These are the systems that want to actually live somewhere and do work over time.

Examples:

- Hermes Agent
- OpenClaw
- other personal-assistant runtimes and control planes

These care about:

- messaging surfaces
- scheduling
- tool routing
- memory
- device access
- operational boundaries

### Layer B: harnesses

These are the batteries-included frameworks for building longer-running agents without wiring every primitive yourself.

Examples:

- Deep Agents
- similar coding/research harnesses

These care about:

- planning
- subagents
- context compression
- execution model
- developer ergonomics

### Layer C: context and memory infrastructure

These focus on the information substrate more than the user-facing agent.

Examples:

- OpenViking
- memory systems and context databases
- retrieval layers and long-term state systems

These care about:

- storage model
- retrieval quality
- observability
- durability
- cost and token efficiency

### Layer D: surfaces and products

These are the interfaces people actually touch.

Examples:

- terminal coding agents
- web UIs
- chat integrations
- domain-specific assistants

This layer often gets the attention, but it’s increasingly downstream of the others.

## 5.5) A cleaner way to compare “better than OpenClaw”

It’s easy to make this conversation dumb.

If you compare every project against OpenClaw as if they all need to be full replacements, you’ll miss what’s actually happening.

A better frame is:

### Better than OpenClaw at **what**?

#### Better at harnessing long tasks
Deep Agents looks stronger.

#### Better at foregrounding self-improvement and procedural skill growth
Hermes looks stronger.

#### Better at context architecture and retrieval structure
OpenViking looks stronger.

#### Better at broad assistant surface area, channels, nodes, and control-plane utility
OpenClaw still looks stronger.

That’s the useful comparison.

Not “who wins?”

More like: **which layer is improving fastest, and who is pushing it forward?**

## 6) What’s actually new in 2026, and what’s just better branding

A lot of this wave is not new in the absolute sense.

Planning, subagents, memory, retrieval, files, and tools have been around.

What’s new is that projects are getting more honest about what’s required for agents to work well outside a demo.

### The real advances

I’d put these in the “real” bucket:

- **context management** becoming explicit rather than accidental
- **subagents for isolation** becoming normal
- **file-backed or externalized state** replacing giant chat blobs
- **persistent memory** being treated as infrastructure instead of a prompt trick
- **scheduling and long-running operation** entering the default design
- **sandboxing / backend separation** becoming part of the architecture conversation

### The less-real advances

I’d be more skeptical of:

- vague claims about “self-improving” without clear curation mechanisms
- frameworks that are mostly prompt packaging pretending to be new science
- anything that says it solved memory just because it added vector search
- agent demos that ignore observability, security, and failure recovery

The category is still full of theatrical language.

But underneath that, some of the engineering is finally sanding down into something practical.

## 7) What builders should actually pay attention to

If you’re building with these tools, I think the important questions are not “which framework is best?”

The better questions are:

### 1. Where does state live?

If the answer is “mostly in chat history,” the system is weaker than it looks.

### 2. How is context managed when work gets long?

Compression, file offloading, tiered loading, and context isolation are not optional anymore.

### 3. What is the memory model?

Not just “do you have memory,” but:

- what gets persisted
- what gets summarized
- what decays
- what is project-local vs user-global
- how recall is debugged

### 4. What is the surface area and blast radius?

An agent that can read messages, run shell, browse the web, edit files, and message people back is not a toy. Treat it like an automation system with risk.

### 5. What part of the stack are you really buying?

Do you need:

- a personal assistant runtime?
- an agent harness?
- a context database?
- a productized coding agent?

Buying the wrong layer is how teams end up frustrated.

## 8) My current verdict on the main players

If I had to compress this into a practical read:

### Deep Agents

**Best described as:** an opinionated harness for long-running autonomous work.

**Why it matters:** it productizes patterns that serious agents need, especially context management and isolated delegation.

**Watch if:** you build research/coding/workflow agents and want a structured base instead of raw primitives.

### Hermes Agent

**Best described as:** a persistent personal-agent runtime with memory, skills, messaging, and scheduling.

**Why it matters:** it’s trying to solve the “agent that actually lives somewhere” problem, not just the “agent that completes a benchmark” problem.

**Watch if:** you care about long-lived assistants, remote operation, and multi-surface personal AI.

### OpenViking

**Best described as:** a context database for agents.

**Why it matters:** it attacks the real bottleneck, context organization and retrieval quality, at the infrastructure level.

**Watch if:** you think memory and context architecture are now more important than prompt wording (I do).

## Bottom line

The agent ecosystem is finally starting to sort itself into recognizable categories.

That’s healthy.

OpenClaw is still one of the most important projects in the whole space because it made the personal-agent control-plane model real for a huge number of builders.

But this update matters because newer projects are now clearly improving on specific parts of the stack.

Deep Agents is pushing the harness pattern forward.
Hermes Agent is pushing the persistent personal-agent model forward.
OpenViking is pushing context infrastructure forward.

Those are different bets, but they rhyme.

The common theme is that the useful agent stack is becoming less about “a clever loop around an LLM” and more about:

- context architecture
- durable state
- isolation
- memory
- scheduling
- surfaces
- operator control

That’s the real shift.

The prompt was never the whole product. Now the tooling is finally starting to admit it.

---

## Sources and references

Primary sources:
- LangChain Deep Agents overview: https://docs.langchain.com/oss/python/deepagents/overview
- LangChain Deep Agents landing page: https://www.langchain.com/deep-agents
- LangChain Deep Agents GitHub: https://github.com/langchain-ai/deepagents
- Nous Hermes Agent docs: https://hermes-agent.nousresearch.com/docs/
- Nous Hermes Agent GitHub: https://github.com/NousResearch/hermes-agent
- OpenViking GitHub: https://github.com/volcengine/OpenViking

X / Bird research signals reviewed for this update:
- LangChain Deep Agents discussion on X via Bird search
- Hermes Agent discussion on X via Bird search
- OpenViking discussion on X via Bird search
- broader “AI agent harness” / viral GitHub chatter on X via Bird search

Notes:
- X is useful here as a signal layer for what is going viral and how people are framing these projects, not as the source of truth for technical claims.
- For factual claims in this post, I weighted official docs/repos above social posts.
