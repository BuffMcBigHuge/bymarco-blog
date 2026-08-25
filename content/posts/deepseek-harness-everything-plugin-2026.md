---
title: "DeepSeek Harness: The Agent Runtime Where Everything Is a Plugin"
date: 2026-08-25T12:00:00-04:00
draft: false
description: "DeepSeek Harness (dsh) is the MIT agent runtime where models, tools, sandboxes, and even the agent loop itself are swappable plugins. Architecture, code mode, honest limits."
summary: "DeepSeek Harness hit 195K stars in two weeks. It is less a coding agent than the machinery a coding agent is assembled from — everything is a plugin, built on Cordis, with append-only session logs and cross-vendor subagents. Here's what's real and what still hurts."
tags:
  - ai
  - agents
  - agentic-ai
  - open-source
  - developer-tools
  - deepseek
  - production
categories:
  - deep-dives
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
ShowRssButtonInSectionTermListPage: true
UseHugoToc: true
cover:
  image: "images/deepseek-harness/overview.png"
  alt: "Modular diagram of the DeepSeek Harness plugin architecture: agent loop, model, tools, sandbox, and session log as swappable hexagon modules"
  caption: "DeepSeek Harness turns the agent runtime into a plugin tree with no privileged core"
  relative: true
  hidden: false
---

Most coding agents share a shape. There's a core loop — take input, call the model, run tools, repeat — written as privileged code, with a ring of extension points around it: MCP servers, skills, hooks. The core is framework-private. Extensions observe rather than participate.

DeepSeek Harness inverts that. It's a runtime where the model adapter, the tool registry, the session log, the sandbox, the UI, even the agent loop itself are all plugins sitting at the same privilege level. Change the loop is a config line, not a fork. The project's one-line pitch — "Everything is a Plugin" — is specific enough to be falsifiable, and it mostly holds.

That's why it's the fastest repo on GitHub. Launched August 13, 2026 as a developer preview, it pulled roughly 92,000 stars in its first 28 hours, 195,000 with 22,000 forks in under two weeks. For reference, OpenClaw — the previous velocity record holder — took 84 days to reach 200,000 stars.

But stars are hype, not adoption. Here's what actually makes this thing different, what breaks, and whether you should run it today.

## The Quick Spec

- **What it is:** an open-source agent harness (`dsh`) from DeepSeek AI, MIT-licensed
- **Install:** `npx @deepseek-ai/dsh web` → Web UI at `http://127.0.0.1:3080`
- **Foundation:** Cordis, a TypeScript plugin framework with four years of production use inside the Koishi chatbot project
- **Language:** TypeScript monorepo, ~500K LOC plus ~300 lines of C11 for the sandbox launcher
- **Release:** v0.1.1-rc.2 (August 21, 2026), explicitly a developer preview
- **Models:** agnostic — DeepSeek's own or ~40 providers, and subagents can even be delegated to a rival vendor's agent
- **The warning, verbatim:** "THERE WILL BE COMPATIBILITY-BREAKING CHANGES"

## What "Everything Is a Plugin" Actually Means

Stop thinking "plugin marketplace." That's the boring half. The slogan is really about turning implementation choices into named, swappable runtime dependencies.

Underneath sits Cordis. Plugins contribute services, typed events, and reversible effects to a shared context; registrations unwind automatically when a plugin unloads. That reversibility is the trick that makes the claim hold. A registration isn't a mutation you hope someone remembers to undo — it's an effect with an inverse the runtime owns. Tool schemas, prompt sections, adapters, and listeners all register this way, so unloading a plugin predictably clean.

The architecture doc is blunt: "There is no privileged core to patch: you extend dsh by mounting a plugin beside the others."

The minimal plugin is a file exporting an `apply(ctx)` function:

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'hello'

export function apply(ctx: Context) {
  console.log('hello from my first plugin')
}
```

Mount it with one line of YAML and it sits at the same privilege level as the model adapter or the tool registry.

A running dsh is a plugin tree composed at boot from ordered layers. A **profile** lists the bundles it stacks and holds your `cordis.patch.yml`; a **bundle** distributes Cordis config rows plus the code they mount. `dsh-base` supplies model adapters, tools, persistence, sandbox, and approval policy. `dsh-web-app` adds the browser; `dsh-headless` is a one-shot runner with no server.

The honesty check on the whole claim is one command:

```sh
dsh --profile web --dump-config
```

Every row it prints is a component you can replace with a patch of your own. If that command showed a hidden privileged core, the marketing would be exposed. It doesn't.

## The Turn/Step Event Loop

The agent loop is not a hard-coded `while` loop. It's decomposed into interceptable events:

```
turn/start → agent/pre-step → step/start
  → assemble prompt sections + tool schemas
  → agent/request → llm/stream → assistant/message
  → tools/pre-execute → tools/execute → tools/post-execute
  → step/end → agent/turn-stopping → turn/end
```

A **step** is one model request plus the tools it calls. A **turn** is zero or more steps. Nearly every stage is interceptable: a plugin can rewrite messages or refuse execution at `agent/pre-step`, or substitute a tool's result at `tools/post-execute`. The loop is a public protocol, not framework-private code.

Event waterfalls are how policy attaches. A listener on `tools/*` calls `next()` to delegate down the chain, or short-circuits to own the decision. That single convention lets permission policy, sandboxing, retries, and metrics all attach to tool execution without any of them knowing about each other.

## Append-Only Session Logs

This is the least controversial strength in the whole project, and maybe the most undervalued.

dsh enforces a runtime invariant: anything that reaches a model request must be reconstructible from the session log. The log is an append-only event stream — `turn/*`, `step/*`, `user/message`, `assistant/*`, `tool/*` — and the history the model sees is projected from it. Resume, fork, search, and replay fall out of that design rather than being bolted on. Context compaction is an independent plugin too, bracketed by three log events so a crash mid-compaction stays reconstructible.

In the Web UI's Trajectory view you watch exactly what reached the wire. The `request/header` event records the config actually sent — which is what one reviewer credits with diagnosing a 3.5x cost blowup in minutes. At a moment when several vendors are encrypting reasoning traces and making agent behavior hard to audit, full traceability as an architectural guarantee is a real differentiator. If you need auditable agent execution records, nothing else in open source does this yet.

## Code Mode and Cross-Vendor Subagents

Two capabilities here genuinely don't exist elsewhere.

**Code mode (`run_code`).** Instead of emitting a dozen individual tool calls that each round-trip through the context window, the model writes TypeScript that batch-calls tools:

```ts
const files = await tools.glob({ pattern: 'src/**/*.ts' })
const hits = []
for (const f of files.slice(0, 50)) {
  const text = await tools.read({ path: f })
  if (text.includes('TODO')) hits.push(f)
}
return hits
```

Only what's printed or returned goes back to the model. A dozen round-trips collapse into one execution, directly attacking the "scattered tool calls burn the context window" problem — a real bill-saver given how token-heavy this harness is.

**Subagents delegable to a competitor.** `ctx.subagents` supports multiple backends, and one can be a rival's agent. You can run dsh as the orchestration layer and hand a subtask to Claude Code or Codex. Very few vendors would ship that.

There's also a set of `cordis_*` tools that let the model inspect the live plugin graph, write, and mount its own plugins at runtime — the seed of a self-evolving agent. Two caveats: they're off by default in every official preset, and a plugin written on the fly lives only in memory. It's gone on restart, with no persistence path yet.

## Sandboxing and the Trust Gap

The security work is not sloppy. Linux uses bwrap + Landlock behind a custom fail-closed C launcher; macOS uses Seatbelt; Windows uses restricted ACL tokens. The approval model is a closed enumeration — anything anomalous is rejected as "unavailable" rather than silently allowed. It even reports enforcement honestly as `full` or `partial`, flagging older Landlock kernels rather than overclaiming.

But there's a layering gap worth flagging loudly. **The sandbox governs tool execution, while plugins themselves run inside the harness process.** Any third-party plugin can reach the shell and filesystem directly. With a plugin repository that crossed 2,000+ submissions in two days, the trust model for community plugins amounts to good faith. Treat a dsh plugin like an npm postinstall script, not like an MCP server.

The sandbox also governs filesystem effects only. Network access and process visibility sit outside its vocabulary — which matters if you're piloting it on code that talks to the outside world.

## The Honest Limits

This is a preview build and it shows.

- **Token usage runs high.** One developer's preliminary comparison found uncached input around 47.6K tokens for dsh versus ~4.5K for Pi on the same model — an order of magnitude. Full pluggability shows up on the bill. The first call in a fresh session carries the whole system prompt, tool schemas, and skill catalogue.
- **A confirmed context-duplication bug.** dsh reads both `CLAUDE.md` and `AGENTS.md`. Many repos keep those files identical for cross-tool compatibility — in which case the instruction set gets injected twice, doubling the system prompt. No official fix at the time of writing.
- **Third-party compatibility is thin.** The official list shows 41 compatible integrations against 219 flagged as needing attention. One hands-on test found all five third-party tools it tried failed outright.
- **The docs are inverted.** Internal architecture docs run to ~170,000 lines with CI checks preventing drift, while user-facing onboarding is thin. Documentation written for agents and documentation for newcomers are different artifacts, and the latter is missing.
- **Config gotchas are buried.** The settings that control cost and correctness live in package READMEs, not the guide. A model entry that declares only `id` inherits a default context window, and the adapter guesses the reasoning dialect from your endpoint's URL — fail that guess and a private DeepSeek gateway gets spoken to in the OpenAI dialect, silently tripling your token bill.
- **Benchmark transparency questions.** V4-Pro's SWE-bench Verified score circulates in two versions: 80.6% self-reported versus 96.4% from third-party evaluator Vals — a 16-point gap with no authoritative explanation. Official agent benchmarks ran in dsh's minimal mode, which raises a fair question about model capability versus framework boost.

## Comparison With Alternatives

Star counts approximate, August 2026:

| Project | Approach | License | Stars | One-liner |
|---|---|---|---|---|
| DeepSeek Harness | Runtime-pluggable | MIT | ~195k | Everything is a plugin; even the loop swaps |
| Claude Code | Layered extensible ecosystem | Source-available | ~141k | Skills/hooks/subagents/MCP — fullest ecosystem |
| OpenAI Codex CLI | Rust + kernel-level sandbox | Apache 2.0 | ~106k | Safety leader; declarative extensions |
| Pi | Minimalist | MIT | ~90k | Four default tools, sub-1K-token system prompt |
| OpenCode | Open ecosystem | Open source | ~197k | Broad provider support, mature TUI |

The most illuminating debate is dsh versus Codex. Codex is declarative: a plugin is a folder on disk — a Markdown skill, an MCP config, a shell script — that never enters the harness process. Reload takes 2-3 seconds and the barrier to entry is near zero. dsh is imperative: plugins carry state, run in-process, and register into each other. Hot-swapping one means handling dangling references and rolling back cleanly — exactly why it needed four years of Cordis underneath.

## Should You Use It?

Two separate questions, two answers.

If you want a coding agent to write production code today: **not this, not yet.** Cost-per-task is high before tuning, the plugin ecosystem is days old, and the API surface is explicitly unstable. Use a pinned version on a contained pilot if you want to explore; don't bet a pipeline you can't babysit.

If you build agent infrastructure — or want to understand where the harness layer is heading: **this has no other working reference implementation.** It's the first time a frontier lab open-sourced its entire harness under MIT, decision records included. That's a shot at the business model: on this reading, the harness should not be a paid product at all. Models, tools, state, permissions, and the loop become an architecture your team can inspect, compose, and test instead of a black box you pay a subscription for.

The verdict that keeps landing is: dsh wins on speed-per-dollar and flexibility, Claude Code wins on absolute quality by a hair, and both reviews stump for running it for an afternoon to learn more than any roundup tells you. I agree with all three. Ten minutes with `--dump-config` will show you the scaffolding every other coding agent hides behind a fence.