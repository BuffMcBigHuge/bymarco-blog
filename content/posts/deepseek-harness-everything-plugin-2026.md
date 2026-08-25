---
title: "DeepSeek Harness: The Agent Runtime Where Everything Is a Plugin"
date: 2026-08-25T12:00:00-04:00
draft: false
description: "DeepSeek Harness (dsh) is the MIT agent runtime where models, tools, sandboxes, and even the agent loop itself are swappable plugins. Architecture, code mode, honest limits."
summary: "DeepSeek Harness hit 195K stars in two weeks. It's less a coding agent than the machinery a coding agent gets assembled from: everything is a plugin, built on Cordis, with append-only session logs and subagents that can delegate to competitors. What's real, what still hurts."
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

Every coding agent you've used has the same skeleton. A core loop sits at the center: take input, call the model, run tools, repeat. Around it, a ring of sanctioned extension points, MCP servers and skills and hooks, all of which get to *observe* the loop and none of which get to *be* the loop. The core is framework-private. You fork if you want to change it.

DeepSeek Harness threw that skeleton out. In this runtime the model adapter, the tool registry, the session log, the sandbox, the UI, and the agent loop itself are all plugins, sitting at exactly the same privilege level. Swapping the loop for a different driver is a config line. Not a fork, a config line.

The one-line pitch, "Everything is a Plugin," is specific enough to be falsifiable. Mostly it holds.

Which brings us to the stars. Launched August 13, 2026 as a developer preview, the repo pulled roughly 92,000 stars in its first 28 hours and passed 195,000 with 22,000 forks inside two weeks. OpenClaw, the previous velocity record holder, needed 84 days to reach 200,000. Stars measure hype, though, not adoption. The interesting question is what DeepSeek actually shipped, and where it hurts when you run it.

## The Quick Spec

- **What it is:** an open-source agent harness (`dsh`) from DeepSeek AI, MIT-licensed
- **Install:** `npx @deepseek-ai/dsh web` → Web UI at `http://127.0.0.1:3080`
- **Foundation:** Cordis, a TypeScript plugin framework with four years of production use inside the Koishi chatbot project
- **Language:** TypeScript monorepo, ~500K LOC plus ~300 lines of C11 for the sandbox launcher
- **Release:** v0.1.1-rc.2 (August 21, 2026), explicitly a developer preview
- **Models:** agnostic. DeepSeek's own or ~40 providers, and subagents can be delegated to a rival vendor's agent
- **The warning, verbatim:** "THERE WILL BE COMPATIBILITY-BREAKING CHANGES"

## What "Everything Is a Plugin" Actually Means

Forget the plugin-marketplace reading. That's the boring half of the slogan. The real claim is that implementation choices become named, swappable runtime dependencies.

Underneath sits Cordis. Plugins contribute services, typed events, and reversible effects to a shared context. When a plugin unloads, its registrations unwind automatically. This reversibility is the whole trick. A registration isn't a mutation someone hopes gets cleaned up later; it's an effect with an inverse the runtime owns. Tool schemas, prompt sections, adapters, listeners, all registered this way, so teardown is predictable instead of leaky.

The architecture doc puts it plainly: "There is no privileged core to patch: you extend dsh by mounting a plugin beside the others."

A minimal plugin is a file exporting an `apply(ctx)` function:

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'hello'

export function apply(ctx: Context) {
  console.log('hello from my first plugin')
}
```

One line of YAML mounts it, and now your hello-world sits at the same privilege level as the model adapter.

A running dsh is a plugin tree composed at boot from ordered layers. A **profile** lists the bundles it stacks and holds your `cordis.patch.yml`. A **bundle** distributes Cordis config rows plus the code they mount. `dsh-base` supplies model adapters, tools, persistence, sandbox, approval policy. `dsh-web-app` adds the browser. `dsh-headless` is a one-shot runner with no server at all.

And there's an honesty check for the entire "no privileged core" claim:

```sh
dsh --profile web --dump-config
```

Every row this prints can be replaced by a patch of your own. If a hidden core existed, this command would be where the marketing fell apart. It doesn't fall apart.

## The Turn/Step Event Loop

There is no hard-coded `while` loop. The agent loop decomposes into interceptable events:

```
turn/start → agent/pre-step → step/start
  → assemble prompt sections + tool schemas
  → agent/request → llm/stream → assistant/message
  → tools/pre-execute → tools/execute → tools/post-execute
  → step/end → agent/turn-stopping → turn/end
```

A **step** is one model request plus the tools it calls. A **turn** is zero or more steps. Nearly every stage is interceptable: a plugin can rewrite messages or refuse execution at `agent/pre-step`, or substitute a tool's result at `tools/post-execute`. The loop is a public protocol rather than framework-private code.

Policy attaches through event waterfalls. A listener on `tools/*` calls `next()` to pass control down the chain, or skips the call and owns the decision itself. That one convention lets permission policy, sandboxing, retries, and metrics attach to tool execution without any of them knowing the others exist.

## Append-Only Session Logs

Of everything in the project, this is the strength I'd bet on.

dsh enforces a runtime invariant: anything that reaches a model request must be reconstructible from the session log. The log is an append-only event stream (`turn/*`, `step/*`, `user/message`, `assistant/*`, `tool/*`), and the history the model sees is projected from it. Resume, fork, search, replay all fall out of that design instead of being bolted on afterward. Even context compaction is an independent plugin, bracketed by three log events so a crash mid-compaction stays reconstructible.

In the Web UI's Trajectory view you watch exactly what reached the wire. One reviewer credits the `request/header` event, which records the config actually sent, with diagnosing a 3.5x cost blowup in minutes instead of hours.

Think about where the industry is going: several vendors are encrypting reasoning traces right now, making agent behavior progressively harder to audit. Making full traceability an architectural guarantee, enforced in code, cuts against that current. If compliance or research requires auditable execution records, nothing else in open source offers this yet.

## Code Mode and Cross-Vendor Subagents

Two capabilities here have no equivalent anywhere else I can find.

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

Only printed output and return values go back to the model. Twelve round-trips become one execution. Given how token-heavy this harness is (more on that below), this matters for the bill, not just elegance.

**Subagents delegable to a competitor.** `ctx.subagents` supports multiple backends, and one of them can be a rival's agent. Run dsh as your orchestration layer and hand a subtask to Claude Code or Codex. I can't think of another vendor that would ship that.

There's also a set of `cordis_*` tools letting the model inspect the live plugin graph, write, and mount its own plugins at runtime. A self-evolving agent, in seed form. Two catches: they're off by default in every official preset, and a plugin written on the fly lives only in memory. Restart wipes it. No persistence path yet.

## Sandboxing and the Trust Gap

Credit where due: the sandbox work is careful. Linux runs bwrap + Landlock behind a custom fail-closed C launcher. macOS uses Seatbelt, Windows restricted ACL tokens. The approval model is a closed enumeration, so anything anomalous gets rejected as "unavailable" instead of silently allowed. Enforcement reports honestly as `full` or `partial`, flagging older Landlock kernels rather than overclaiming.

Now the gap. **The sandbox governs tool execution. Plugins themselves run inside the harness process**, outside it. Any third-party plugin can reach the shell and filesystem directly. The plugin repository crossed 2,000 submissions within two days of launch, and the trust model for all that community code amounts to good faith. Treat a dsh plugin like an npm postinstall script, not like an MCP server.

One more limit while we're here: the sandbox covers filesystem effects only. Network access and process visibility sit outside its vocabulary. If your pilot talks to the outside world, the sandbox is not what's protecting it.

## The Honest Limits

It's a preview build, and it shows.

Token usage runs high. One developer's preliminary comparison found uncached input around 47.6K tokens for dsh versus roughly 4.5K for Pi on the same model. An order of magnitude. Every fresh session pays for the full system prompt, tool schemas, and skill catalogue up front, which is the price of shipping everything pluggable.

There's a confirmed context-duplication bug. dsh reads both `CLAUDE.md` and `AGENTS.md`, and many repos keep those files identical for cross-tool compatibility. Identical files means the instruction set gets injected twice and your system prompt doubles. No official fix at time of writing.

Third-party compatibility is thin. The official list shows 41 compatible integrations against 219 flagged as needing attention. One hands-on test tried five third-party tools; all five failed outright.

The docs are inverted. Internal architecture documentation runs to about 170,000 lines with CI checks preventing drift against source, while user-facing onboarding barely exists. Docs written for agents and docs written for newcomers are different artifacts. Only one got written.

Some cost-critical settings are buried in package READMEs instead of the guide. Declare a model entry with only an `id` and it inherits default context-window fallbacks. Worse, the adapter guesses your endpoint's reasoning dialect from its hostname. Run a DeepSeek-dialect gateway on a neutral URL and the harness speaks OpenAI dialect to it, silently tripling token spend on some workloads. The fix exists (`compat.thinkingFormat`), finding it does not.

Benchmark transparency deserves its own paragraph. V4-Pro's SWE-bench Verified score circulates as two numbers: 80.6% self-reported versus 96.4% from third-party evaluator Vals. Sixteen points apart, no authoritative explanation for the gap. Official agent benchmarks ran in dsh's minimal mode, which raises a fair question about how much score comes from the model versus the framework. To be precise about the accusation: nobody has publicly reproduced and disproven any official number. The complaint is transparency, not fabrication. And an open-source harness is what makes independent reproduction possible in the first place, which is arguably the point of shipping it.

## Comparison With Alternatives

Star counts approximate, August 2026:

| Project | Approach | License | Stars | One-liner |
|---|---|---|---|---|
| DeepSeek Harness | Runtime-pluggable | MIT | ~195k | Everything is a plugin; even the loop swaps |
| Claude Code | Layered extensible ecosystem | Source-available | ~141k | Skills/hooks/subagents/MCP — fullest ecosystem |
| OpenAI Codex CLI | Rust + kernel-level sandbox | Apache 2.0 | ~106k | Safety leader; declarative extensions |
| Pi | Minimalist | MIT | ~90k | Four default tools, sub-1K-token system prompt |
| OpenCode | Open ecosystem | Open source | ~197k | Broad provider support, mature TUI |

The comparison I find most instructive is dsh against Codex, because they sit at opposite ends of one axis. Codex extensions are declarative: a plugin is a folder on disk holding a Markdown skill or an MCP config, and it never enters the harness process. Reload takes two or three seconds and anyone can write one. dsh plugins are imperative: they carry state, run in-process, register into each other. Hot-swapping one at runtime means handling dangling references and rolling back cleanly from a crash, which is exactly the problem Cordis spent four years solving. Declarative wins on accessibility. Imperative wins on power. Pick your poison knowingly.

## Should You Use It?

Depends which of two people you are.

**You want an agent to write production code today:** don't use this one. Cost per task is high before tuning, the plugin ecosystem is weeks old, and the API surface is unstable by the project's own announcement. Pin a version and run a contained pilot if you're curious. Don't put a pipeline on it that you can't babysit.

**You build agent infrastructure:** run it this week. This is the first time a frontier lab open-sourced its entire harness layer under MIT, decision records included, and there's no other working reference implementation of the approach. Read as a business move, it argues the harness should never have been a paid product at all. Models, tools, state, permissions, the loop: an architecture your team can inspect and swap instead of a black box on a subscription.

Ten minutes with `--dump-config` will show you the scaffolding every other coding agent hides behind a fence.