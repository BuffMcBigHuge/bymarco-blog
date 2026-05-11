---
title: "Speculative Decoding Is Finally Useful for Local LLMs"
date: 2026-05-07T02:10:00-04:00
lastmod: 2026-05-11T14:45:00-04:00
draft: false
slug: "speculative-decoding-local-llms-2026"
description: "A practical field guide to speculative decoding for local LLMs, updated with new llama.cpp MTP, TurboQuant HIP, and BeeLlama.cpp field reports for Qwen3.6 on RTX 4090, RTX 3090, and Radeon 7900 XTX."
summary: "Speculative decoding has moved from inference-paper trivia into the local LLM hot path. The useful version now includes ngram speculation, native MTP heads, DFlash-style drafting, and TurboQuant / TCQ KV compression, with early llama.cpp forks showing both real speedups and real rough edges."
tags:
  - ai
  - local-llm
  - inference
  - speculative-decoding
  - llama-cpp
  - vllm
  - qwen
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
  image: "/images/posts/speculative-decoding-local-llms-2026/hero.svg"
  alt: "Speculative decoding pipeline showing draft, verify, and commit stages for local LLM inference"
  caption: "Speculative decoding is becoming the practical speed layer for local models."
  relative: false
  hidden: false
---

Local LLM speed used to be a hardware conversation.

Buy more VRAM. Use a smaller quant. Accept worse quality. Wait longer.

That is still part of the story, but the story has widened.

The more interesting thing happening in May 2026 is that **decoding strategy** is starting to matter as much as model size.

Speculative decoding has been around for years, but it is finally crossing the line from “cool paper” to “I should probably turn this on for my local coding model.”

The reason is simple: the local stack now has 3 usable speed layers.

- **ngram speculation** in llama.cpp for cheap wins on repeated text
- **MTP** in models like Qwen3.6, where the model ships with native multi-token prediction heads
- **TurboQuant / KV compression** in serving stacks like vLLM, where long context is the memory bottleneck

![Three speed layers for local decoding](/images/posts/speculative-decoding-local-llms-2026/stack.svg)

The short version: speculative decoding is becoming one of the main reasons local agents will feel less like toys.

## The short version

Speculative decoding is a way to make an LLM generate faster without simply making the model smaller.

A cheap drafter proposes future tokens. The real model verifies them. If the guesses are right, you get multiple tokens out of one expensive pass. If the guesses are wrong, the system falls back safely.

That verification step matters.

Correctly implemented speculative decoding should not make the model “dumber” by itself. It should preserve the target model’s output distribution while reducing latency. The danger comes from rough implementations, bad quantization, bad templates, over-aggressive token counts, or serving bugs.

That is exactly why the current local LLM conversation is so useful. People are not just repeating benchmark slides. They are finding the sharp edges.

The pattern from Bird/X and r/LocalLLaMA is pretty clear:

- Qwen3.6-27B with MTP is the current hero setup
- vLLM has practical MTP paths for Qwen-family models
- llama.cpp has merged ngram-mod and speculative checkpointing, while native MTP support is still being worked through in an open PR
- Local users are reporting 2x-ish wins in favorable Qwen3.6 MTP setups, with bigger or smaller gains depending on hardware, quant, context length, and workload
- TurboQuant is exciting for long-context memory, but recent vLLM bug reports show that TurboQuant plus speculative decoding needs careful validation

The hype is justified.

The defaults are not settled.

**May 11 update:** the local stack moved again. Three new community paths are worth separating from the original llama.cpp/vLLM discussion: Indras-Mirror's CUDA MTP plus fused TBQ4 fork for RTX 4090-class cards, domvox's HIP/ROCm TurboQuant fork for RDNA3 cards like the 7900 XTX, and Anbeeld's BeeLlama.cpp fork, which packages DFlash, adaptive drafting, TurboQuant/TCQ cache types, multimodal support, and reasoning-loop protection into a more productized llama.cpp variant.

{{< linkcard
  url="https://github.com/Indras-Mirror/llama.cpp-mtp"
  title="llama.cpp-mtp: Fused TBQ4 Flash Attention + MTP + Shared Tensors"
  site="GitHub"
  author="Indras-Mirror"
>}}

{{< linkcard
  url="https://github.com/domvox/llama.cpp-turboquant-hip"
  title="llama.cpp-turboquant-hip: TurboQuant KV cache compression for HIP/ROCm"
  site="GitHub"
  author="domvox"
>}}

{{< linkcard
  url="https://github.com/Anbeeld/beellama.cpp"
  title="BeeLlama.cpp: DFlash, TurboQuant, TCQ, and multimodal llama.cpp fork"
  site="GitHub"
  author="Anbeeld"
>}}

## Why speculative decoding matters more locally than in the cloud

Cloud inference providers can hide a lot behind fleet-level optimization.

Local inference cannot.

If you are running a 27B or 35B model on a 3090, 4090, 5090, RTX PRO card, Strix Halo, or Apple Silicon box, every millisecond is visible. The difference between 22 tokens per second and 55 tokens per second is not academic. It changes whether an agent loop feels alive.

That matters most for coding agents.

A chat answer can be slow. A coding agent cannot.

Coding agents read files, produce diffs, run tests, inspect logs, repair, and repeat. They generate a lot of highly patterned text: code, JSON, XML, markdown, shell output, stack traces, file paths, import blocks, boilerplate, and patches.

That is exactly the kind of workload where speculative techniques can work well.

## How speculative decoding works

The clean mental model is:

1. **Draft**: produce candidate future tokens cheaply
2. **Verify**: ask the main model whether those tokens match what it would have produced
3. **Commit**: accept the matching prefix and continue

![Speculative decoding tradeoffs](/images/posts/speculative-decoding-local-llms-2026/tradeoffs.svg)

The speedup depends on acceptance rate.

If the draft is usually right, you win big. If it is usually wrong, you pay overhead and get little back.

That is why speculative decoding is better understood as a family of draft-and-verify techniques.

## 1) ngram speculation: the cheap win

ngram speculation is the least glamorous version and probably the easiest one to underestimate.

It does not require a second model. It does not require native MTP heads. It does not require a special checkpoint.

It looks for repeated token sequences and predicts that the next span will follow a pattern already seen in the prompt or generated output.

That sounds too dumb to matter, until you remember what local coding workloads look like:

- repeated function names
- repeated imports
- repeated indentation
- repeated JSON keys
- repeated markdown structure
- repeated file paths
- repeated log lines
- repeated boilerplate in generated patches

For code and structured text, repetition is everywhere.

llama.cpp merged `spec : add ngram-mod` in PR #19164 on January 30, 2026. Then speculative checkpointing landed in PR #19493 on April 19, 2026, improving the server-side path for these techniques.

{{< linkcard
  url="https://github.com/ggml-org/llama.cpp/pull/19164"
  title="llama.cpp PR #19164: spec: add ngram-mod"
  site="GitHub"
  author="ggerganov"
>}}

{{< linkcard
  url="https://github.com/ggml-org/llama.cpp/pull/19493"
  title="llama.cpp PR #19493: server: speculative checkpointing"
  site="GitHub"
  author="srogmann"
>}}

This is the boring optimization that becomes powerful because it is broadly applicable.

No second model. No extra memory for a drafter. No fine-tuned head. Just exploit the fact that local agent workloads repeat themselves.

The caveat: ngram speculation is workload-sensitive.

It can look magical on code and templated text, then much less impressive on open-ended prose. That is not a failure. That is the nature of the trick.

## 2) MTP: when the model drafts its own future

MTP means multi-token prediction.

Instead of training only on “predict the next token,” a model can be trained to predict several future tokens. At inference time, those extra heads become a native drafter.

This is why Qwen3.6 is all over the current local LLM conversation.

Qwen3.6 models are landing in a moment where inference engines are ready to use those heads. vLLM has Qwen MTP support paths, and the community is pushing llama.cpp support forward in PR #22673, which is still open as I write this on May 7, 2026.

{{< linkcard
  url="https://github.com/ggml-org/llama.cpp/pull/22673"
  title="llama.cpp PR #22673: llama + spec: MTP Support"
  site="GitHub"
  author="am17an"
>}}

{{< linkcard
  url="https://docs.vllm.ai/en/latest/features/spec_decode.html"
  title="vLLM docs: Speculative decoding"
  site="vLLM Docs"
  author="vLLM"
>}}

On X and Reddit, people are reporting Qwen3.6-27B and Qwen3.6-35B-A3B results that would have sounded fake a year ago: 2x-ish speedups on a single L40S, 50+ tokens per second on dual 3090 rigs, 80+ tokens per second at very long context on a 4090-class setup, and vLLM / llama.cpp configurations where small batching, cache, and draft-depth changes move throughput materially.

{{< linkcard
  url="https://x.com/CDerinbogaz/status/2048800691339039147"
  title="Qwen3.6-27B FP8 with MTP on a single L40S"
  site="X"
  author="Jay Derinbogaz"
>}}

{{< linkcard
  url="https://x.com/Hikari_07_jp/status/2052107591250243777"
  title="Qwen3.6 NVFP4 vs FP8 with MTP on RTX PRO 6000 Blackwell"
  site="X"
  author="Hikari"
>}}

The most important thing in those reports is the shape of the curve.

Dense Qwen3.6-27B plus MTP is starting to look like a serious local coding-agent model because the latency is finally usable.

That changes the product surface. If local inference is fast enough, you can run more loops, keep more experiments local, and treat failure as cheap.

### The new llama.cpp fork layer

The May 11 wave adds an important nuance: **the best numbers are no longer coming from one clean upstream path.** They are coming from forks that combine several optimizations at once.

Indras-Mirror's fork keeps the native Qwen3.6 MTP path, then attacks the KV-cache side with fused TBQ4 flash attention. The interesting claim is architectural, not just benchmark-shaped: instead of dequantizing TurboQuant KV into an intermediate F16 buffer and then feeding flash attention, the fork reads raw TBQ4 blocks inside the flash-attention path and does centroid lookup inline. It also adds shared tensor plumbing to avoid duplicating hundreds of MiB of embedding memory between trunk and MTP models.

The reported setup is very specific: Qwen3.6-27B Heretic-style Q4_K_M with preserved/grafted MTP heads, RTX 4090 24GB, 262k context, TBQ4_0 KV cache, MTP draft 3. The Reddit field report started around 80 to 87 tokens per second with roughly 73% draft acceptance; the repo README later reports a newer May 11 benchmark at 179.4 tokens per second, 81.4% acceptance, and roughly 20GB VRAM for 262k context.

That number is exciting, but I would not treat it as a universal llama.cpp baseline. It is a fast-moving fork, on a specific CUDA card, with specific quantization and MTP-preserved GGUFs. The useful takeaway is narrower and more durable: fusing quantized-KV dequant into the attention hot path is probably the right direction if long context and MTP are going to coexist on 24GB cards.

{{< linkcard
  url="https://www.reddit.com/r/LocalLLaMA/comments/1t7kyju/got_mtp_turboquant_running_qwen3627b_80_ts_at/"
  title="Got MTP + TurboQuant running — Qwen3.6-27B, 80+ t/s at 262K context on RTX 4090"
  site="Reddit"
  author="r/LocalLLaMA"
>}}

BeeLlama.cpp is a different bet. It is not just “MTP in llama.cpp.” It packages DFlash speculative decoding, adaptive draft control, TurboQuant/TCQ KV cache compression, CopySpec, optional branch verification, multimodal support, and reasoning-loop protection into one fork. The README reports Qwen3.6-27B Q4/Q5 DFlash best cases on a single RTX 3090 around 2.3x to 3.7x speedup, with peak generation above 130 tokens per second on simple code-generation tasks.

The important product detail is adaptive drafting. Fixed draft depth is brittle: draft too shallow and you leave speed on the table; draft too deep and low acceptance turns speculation into overhead. Bee's controller tries to adjust draft depth based on whether speculation is beating the no-spec baseline. That is the kind of boring control loop that real local-agent runtimes will need.

{{< linkcard
  url="https://www.reddit.com/r/LocalLLM/comments/1t8905x/beellamacpp_advanced_dflash_turboquant_with/"
  title="BeeLlama.cpp: DFlash, TurboQuant, reasoning and vision on Qwen3.6 27B"
  site="Reddit"
  author="r/LocalLLM"
>}}

## What the Reddit threads add

The Reddit threads are more useful than benchmark screenshots because they include the annoying parts: build failures, weird VRAM spikes, acceptance-rate collapses, tool-calling anxiety, and arguments about whether compressed KV cache is actually preserving quality.

A few patterns stood out.

First, Qwen3.6-27B is still the center of gravity. Earlier threads framed it as roughly 2.5x faster with MTP, but the newer reports are more concrete: exact `llama-server` commands, cache types, draft depths, context sizes, and hardware-specific failures.

{{< linkcard
  url="https://www.reddit.com/r/LocalLLaMA/comments/1t57xuu/25x_faster_inference_with_qwen_36_27b_using_mtp/"
  title="2.5x faster inference with Qwen 3.6 27B using MTP"
  site="Reddit"
  author="r/LocalLLaMA"
>}}

Second, the 4090 path is ahead of the cleaner upstream story. The Indras-Mirror report combines Qwen3.6-27B, preserved/grafted MTP heads, `--spec-type mtp`, draft depth 3, `tbq4_0` for K/V cache, and 262k context. The post claims 80 to 87 tokens per second after optimization and about 73% draft acceptance. In comments, the useful pushback is not “this is fake.” It is sharper: TurboQuant's “lossless” language is contested, Q4 model quality is workload-dependent, and tokens per second is not enough unless the setup survives code, tools, long context, and quality checks.

{{< linkcard
  url="https://www.reddit.com/r/LocalLLaMA/comments/1t7kyju/got_mtp_turboquant_running_qwen3627b_80_ts_at/"
  title="Got MTP + TurboQuant running — Qwen3.6-27B, 80+ t/s at 262K context on RTX 4090"
  site="Reddit"
  author="r/LocalLLaMA"
>}}

Third, AMD is not a footnote. The 7900 XTX thread is exactly the kind of report local inference needs: messy backend-by-backend comparison instead of one heroic chart. The headline result was that Vulkan plus MTP on Qwen3.6-27B reached about 81 tokens per second in the poster's llama-cli test, while ROCm plus MTP was also materially faster than non-MTP but hit an unexplained VRAM spike that limited practical context. ROCm plus TurboQuant remained interesting because of long-context memory, not raw decode speed.

{{< linkcard
  url="https://www.reddit.com/r/LocalLLM/comments/1ta42t1/llamacpp_turboquant_mtp_on_7900_xtx/"
  title="llama.cpp TurboQuant + MTP on Radeon 7900 XTX"
  site="Reddit"
  author="r/LocalLLM"
>}}

That distinction matters for AMD users: **Vulkan may currently be the better MTP speed path, while ROCm/HIP may be the better TurboQuant experimentation path.** The domvox fork exists because the HIP/RDNA3 story needs direct work, not just CUDA ports with the names changed.

Fourth, BeeLlama.cpp is trying to make this less hand-assembled. The thread's value is not only the Qwen3.6 27B Q5 plus 200k-context claim on a 3090. It is the feature bundling: DFlash drafting, adaptive draft-max, TurboQuant/TCQ cache types, multimodal support, reasoning-loop protection, and OpenAI-compatible server flow. The comments also show the current reality: initial Linux build issues got fixed, users hit runtime crashes around grammar/tool-call paths, and low acceptance-rate reports still require full config-level debugging.

{{< linkcard
  url="https://www.reddit.com/r/LocalLLM/comments/1t8905x/beellamacpp_advanced_dflash_turboquant_with/"
  title="BeeLlama.cpp: DFlash, TurboQuant, reasoning and vision on Qwen3.6 27B"
  site="Reddit"
  author="r/LocalLLM"
>}}

Fifth, MoE MTP is still more complicated than dense Qwen3.6-27B.

In a thread about Qwen3.6-35B-A3B with MTP grafted into Unsloth UD XL GGUFs, users were seeing mixed results. Some saw big jumps on specific hardware. Others pointed out that MoE models can benefit less cleanly because the main model still needs to verify the draft, and sparse routing changes the cost profile.

{{< linkcard
  url="https://www.reddit.com/r/LocalLLaMA/comments/1t5r4tz/uploaded_unsloth_qwen3635ba3b_ud_xl_models_with/"
  title="Uploaded Unsloth Qwen3.6-35B-A3B UD XL models with MTP grafted"
  site="Reddit"
  author="r/LocalLLaMA"
>}}

Sixth, quality anxiety is real, but slightly misdirected.

One LocalLLaMA thread asked whether MTP changes intelligence. The best answer is: correctly verified MTP should preserve the target model behavior. The bigger risk lives in the surrounding stack: templates, quantization, runtime bugs, cache compression, sampler settings, aggressive draft depth, and edge cases in grammars or tool calls.

{{< linkcard
  url="https://www.reddit.com/r/LocalLLaMA/comments/1t5v8n0/quality_intelligence_testing_on_mtp/"
  title="Quality testing on MTP"
  site="Reddit"
  author="r/LocalLLaMA"
>}}

The local community is converging on a useful norm: measure acceptance rate, prefill cost, decode speed, VRAM, long-context behavior, and task quality separately. Tokens per second alone is too blunt.

## 3) TurboQuant: the memory side of the same story

TurboQuant belongs in this conversation because local long-context inference is often memory-bound before it is compute-bound.

The KV cache gets ugly fast. If you want 100k, 200k, or 262k context on a local box, the model weights are only one part of the problem. The cache becomes the second model living in your VRAM.

TurboQuant-style KV compression attacks that bottleneck.

Google Research’s TurboQuant work is about compressing KV cache memory while preserving generation quality. vLLM has been exposing KV cache quantization paths, and the local community is starting to combine KV compression with MTP and ngram speculation.

{{< linkcard
  url="https://research.google/blog/turboquant-taming-stateful-memory-costs-for-llm-inference/"
  title="TurboQuant: Taming Stateful Memory Costs for LLM Inference"
  site="Google Research"
  author="Google Research"
>}}

{{< linkcard
  url="https://docs.vllm.ai/en/latest/features/quantization/quantized_kvcache.html"
  title="vLLM docs: Quantized KV Cache"
  site="vLLM Docs"
  author="vLLM"
>}}

This is where the stack gets spicy.

Speculative decoding wants room to run. Long context wants KV cache memory. Quantization wants to reduce memory pressure. Put them together and you get exactly the kind of system where one bad kernel path can ruin your day.

A recent vLLM issue reported degenerate token loops when TurboQuant KV was combined with speculative decoding methods like MTP or ngram. The issue was closed, but the lesson survives: treat TurboQuant plus speculation as a power-user path until you have tested your exact model, quant, context length, and tool-calling format.

{{< linkcard
  url="https://github.com/vllm-project/vllm/issues/40831"
  title="vLLM issue #40831: TurboQuant KV plus speculative decoding token loops"
  site="GitHub"
  author="vLLM"
>}}

This is the correct level of paranoia: verification before trust.

## The practical stack I would try first

If I were setting up a local coding model today, I would split the options by tolerance for pain.

### Low-risk path: llama.cpp ngram

Start with llama.cpp and `ngram-mod` speculation on a model you already trust.

This is the least invasive route.

Use it for:

- coding
- patch generation
- markdown editing
- JSON-heavy agent traces
- repetitive CLI output

Expect uneven gains. That is fine. The setup cost is low enough that even modest wins are worth it.

### Higher-upside path: Qwen3.6 MTP

If you want the current fast local-agent setup, Qwen3.6-27B plus MTP is the thing to test.

A typical vLLM direction looks like Qwen-family MTP speculative config with a small number of speculative tokens, then careful tuning around batching, context, KV cache, and tool calling. A typical llama.cpp direction now means trying the upstream MTP PR or one of the fast-moving forks, depending on whether you care more about mainline safety, CUDA speed, AMD support, or DFlash features.

The trap is over-tuning for tokens per second.

For an agent, I care more about:

- time to first useful edit
- whether tool calls stay valid
- whether JSON stays valid
- whether the model loops at high context
- whether prefill gets worse enough to erase decode gains
- whether acceptance rate holds on real repo tasks

### Fork path: BeeLlama.cpp / fused-TBQ4 llama.cpp

If you are comfortable running forks, there are now 2 different reasons to leave the mainline path.

Use Indras-Mirror-style MTP plus fused TBQ4 if your priority is long-context CUDA performance on a 24GB NVIDIA card and you are willing to track a fast-moving branch.

Use BeeLlama.cpp if you want a more bundled speculative runtime: DFlash, adaptive drafting, TurboQuant/TCQ cache types, multimodal support, CopySpec, and reasoning-loop protection in one llama.cpp-shaped server.

For either path, pin the commit, keep the exact launch command in your notes, and test tool calls. These forks are promising, but they are not boring infrastructure yet.

### AMD path: Vulkan for MTP, HIP for TurboQuant experiments

For Radeon 7900 XTX-class hardware, I would test Vulkan plus MTP first if raw decode speed is the goal.

The current field report suggests Vulkan MTP can be substantially faster than non-MTP and avoids some of the ROCm/MTP memory weirdness. ROCm/HIP is still worth tracking because the domvox TurboQuant fork is directly aimed at RDNA3 KV-cache compression, but the reported VRAM spike around ROCm plus MTP is exactly the kind of issue that can turn a good benchmark into a bad daily driver.

### Experimental path: MTP plus TurboQuant

This is where the upside is obvious and the bugs are plausible.

Use it if:

- you need very long context
- you are already comfortable debugging vLLM or llama.cpp
- you can run a real eval suite before trusting it
- you are willing to pin versions and roll back

Do not use it blindly for a production-ish agent harness.

The failure mode is annoying: it can look fast until tool calling, long context, or a specific prompt template triggers loops.

## What to benchmark

Do not benchmark only “tokens per second on a joke prompt.”

That is how people fool themselves.

For local agents, use a small benchmark pack that matches your actual workload:

1. **Cold prompt**: new repo, large context, no repetition
2. **Patch generation**: edit a real file with repeated identifiers
3. **JSON tool call**: force strict schema output
4. **Long-context repair**: ask it to fix a bug after 50k to 150k tokens of repo context
5. **Markdown rewrite**: repetitive prose and headings
6. **Loop test**: run the same tool-calling prompt 20 times and check for degenerate loops
7. **Quality check**: compare diffs, tests, and hallucinated APIs, not just speed
8. **Grammar/tool-call stress**: run structured outputs while speculation is active
9. **VRAM spike test**: start fresh conversations at several context sizes and watch transient allocations

Track these separately:

- prefill tokens per second
- decode tokens per second
- acceptance rate
- VRAM usage
- context cliff
- tool-call validity
- grammar / JSON crash rate
- transient VRAM spikes
- test pass rate

The best speculative setup is not always the one with the prettiest decode number.

It is the one that makes the whole loop finish faster without making the agent weird.

## The big take

The local LLM story used to be dominated by model releases.

Now the inference layer is getting interesting again.

ngram speculation makes boring repeated text cheap. MTP gives models like Qwen3.6 a native way to draft ahead. DFlash-style drafters push speculation into a more configurable runtime layer. TurboQuant and TCQ give long-context setups more breathing room. Together, they point at a local-agent stack that is much faster without automatically abandoning model quality.

But the May 11 field reports also make the risk clearer. The fast path is no longer one flag. It is model quant, MTP heads, draft depth, KV cache type, backend, flash-attention path, prompt format, and tool-call behavior all interacting at once.

The mistake is treating this as one magic flag.

The better framing is: **speculative decoding is becoming an inference design space.**

That design space is going to matter a lot for local agents.

If a local coding model can run at useful speed, keep 100k+ context, preserve tool calls, and stay cheap enough to loop all day, the economics change.

Local inference becomes a serious way to run always-on agents without paying frontier API prices for every failed attempt.

That is the part worth watching.
