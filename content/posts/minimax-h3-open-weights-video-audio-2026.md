---
title: "MiniMax H3: The Open-Weight Video Model That Finally Comes With Sound"
date: 2026-08-11T12:00:00-04:00
draft: false
description: "MiniMax H3 (Hailuo 3.0) generates 15s 2K video with native stereo audio from text, images, video, and audio refs. How it works, what it costs, community verdict."
summary: "MiniMax H3 is a 33B open-weights model that reads text, images, video, and audio in one context, then outputs 2K video with synced stereo sound. Architecture, official demo clips, and the Reddit/X verdict."
tags:
  - ai
  - video-generation
  - open-source
  - multimodal
  - audio
  - machine-learning
  - comfyui
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
  image: "images/minimax-h3/overview.png"
  alt: "Diagram of the MiniMax H3 three-stage pipeline: context understanding, base generation at 768p, and in-context regeneration to 2K"
  caption: "The H3 system: hosted context understanding, an open H3-Base, and a 2K in-context regeneration stage"
  relative: true
  hidden: false
---

MiniMax H3 (Hailuo 3.0) launched on July 31, 2026, and the open-weights release followed days later. It is the first open video model where the audio is not an afterthought. One transformer reads text, images, video, and audio as a single context, then generates a video with dialogue, sound effects, and music baked into one native stereo track. Up to 15 seconds, up to 2K, at 24 fps.

The reaction was immediate. On r/StableDiffusion, a clip of Will Smith eating spaghetti, the meme that defined the awkward early days of AI video, became the "new benchmark." One top comment on the Hugging Face release thread: "we have never had a model like this." The Hugging Face page already shows 59,000 downloads over the last 30 days, 49 community fine-tunes, and 42 quantizations. That is not hype noise. Something real shipped.

## What MiniMax H3 Is

H3 is a general-purpose, omni-modal generative system. The output spec alone puts it ahead of most closed competitors:

- Duration: 4 to 15 seconds per generation
- Resolution: 768p short side by default; 2K through the H3-Regenerate-2K stage
- Frame rate: 24 fps
- Audio: 32 kHz native stereo, with dialogue, foley, and music modeled jointly
- Languages: stable support for 11, including English, Chinese, Japanese, Korean, Spanish, French, German, Italian, Portuguese, Russian, and Arabic
- Aspect ratios: 21:9 down to 9:16

It ships as two task-specific checkpoints. **H3-Base-FL2VA** handles text-to-video plus first-frame, last-frame, and first-and-last-frame conditioning. **H3-Base-Ref2VA** is the omni-reference variant: up to 9 images, 3 video clips, and 3 audio clips in one request (12 files max across types, each clip 2 to 15 seconds). Want a character to look like Image 2, move like Video 1, and speak with the voice from Audio 3? That is a single Ref2VA call.

## Why One Model Instead of Ten

Image generation has been split into T2I, editing, subject reference, motion reference, and style reference models. Audio has treated voice, sound effects, and music as separate domains. Video has fragmented into text-to-video, image-to-video, first-and-last-frame, video editing, and voice cloning. MiniMax's argument, spelled out in the launch post, is that these silos cap generalization. H3 was trained from the start on all of these tasks together, with reference and editing relationships expressed in natural language. Language, in their words, is the bridge: "Reference the Hitchcock camera movement from Video 1, have the character in Image 2 sing, with the vocals matching Audio 3."

That framing holds up in practice. The model card's own Ref2VA demo is a video edit where a man holding a lamb in a field is re-animated to speak new dialogue, with a second audio clip supplying the voice timbre and the original track kept as background music. That single generation covers video editing, lip-sync, voice cloning, and audio mixing.

## How the System Fits Together

The full product is three modules, and only one of them is open today.

![The three-stage H3 system: H3-Context-IR, H3-Base, H3-Regenerate-2K](/images/minimax-h3/overview.png)

1. **H3-Context-IR** is a hosted preprocessing system that parses the multimodal inputs and rewrites them into a structured prompt. MiniMax says source material runs through roughly 100K tokens of inference, distilled to an average of 4K tokens. It is not in the open release; you either use their API or build your own prompt pipeline.
2. **H3-Base** is the open-weights generator. It produces the 768p result with synchronized stereo audio.
3. **H3-Regenerate-2K** feeds the 768p output plus the original context back into H3 to regenerate at 2K. MiniMax calls this in-context regeneration, and it is the clever part: instead of a generic upscaler guessing at fine detail, the model re-reads the source context and re-renders, which is why small text and brand marks survive the resolution jump. Also not open yet.

## Inside the Transformer

The architecture is refreshingly plain. A Qwen3-VL-32B encoder (layer-50 hidden states) produces text and visual-semantic tokens. A visual VAE compresses video at 16x spatial and 4x temporal with 24 latent channels, then patches 1x2x2 (time, height, width) into the sequence, for an effective 32x spatial compression. An audio VAE turns 32 kHz stereo into 40 Hz tokens per channel. Everything gets packed into one sequence with three-dimensional rotary position embeddings (MM-RoPE) over time, height, and width, and a 33B single-stream dense transformer denoises video and audio latents jointly.

![The H3-Base architecture: modality encoders, packed in-context sequence, 33B omni transformer, and joint decode](/images/minimax-h3/full-arch.png)

A few details worth knowing:

- About 13B of the 33B parameters live in AdaLN modulation branches. Those outputs can be precomputed and cached, so inference-only deployments do not need to load them. The full weights are released for fine-tuning.
- The 50 shared DiT blocks are modality-agnostic. Modality-specific structure is confined to input/output layers and AdaLN. That is the design bet: one backbone, no per-modality experts.
- H3 was trained with native sparse attention (MoBA-style block selection, not the MSA used in MiniMax M3), but the open release runs full attention. A sparse-attention release is promised for faster inference.
- The visual VAE's compression gives an effective 4x gain in sequence length, which is the trick that makes native 2K training affordable.

## The Official Demos

These clips are the official assets from the MiniMax H3 Hugging Face repository, self-hosted here. The first pair is the same prompt at 768p and 2K, so you can judge the regeneration stage with your own eyes.

**Text-to-video, 768p:** a starship bridge, a fleet charging hyperdrives, a jump to lightspeed. Prompt-driven camera cuts, and the score, the hum of life support, and the bass-heavy boom are all generated, not added in post.

<video controls preload="metadata" style="width:100%;border-radius:8px">
  <source src="/images/minimax-h3/t2va.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

**The same prompt regenerated at 2K.** The difference shows up in the detail: individual hair strands, fabric weave, and skin texture stay sharp instead of smoothing out. This is what in-context regeneration buys over a generic upscaler.

<video controls preload="metadata" style="width:100%;border-radius:8px">
  <source src="/images/minimax-h3/t2va_2k.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

**First-frame image-to-video:** a single reference image of a ramen bowl becomes an 8-second family dinner scene with a controlled focus pull, rising steam, and a soundscape of chopsticks, ceramic clinks, and a koto-accompanied score.

<video controls preload="metadata" style="width:100%;border-radius:8px">
  <source src="/images/minimax-h3/fl2va.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

**Reference-to-video:** the multi-modal edit demo. A man in a pink suit holding a lamb is re-animated to speak new lines, with one audio clip supplying the voice timbre and the original music track reused underneath. The dialogue is synced, the identity holds, and the golden-hour lighting never shifts.

<video controls preload="metadata" style="width:100%;border-radius:8px">
  <source src="/images/minimax-h3/ref2va.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

## What the Community Is Doing With It

The launch week produced a predictable gold rush. A fan assembled a full AI Star Trek: The Next Generation scene in ComfyUI. Someone made a Seinfeld sketch where the characters realize they are AI, shot at 1080p with a raw text-to-video workflow. Another user demoed character swaps into existing footage. And the benchmark that keeps getting cited is the Will Smith spaghetti clip, a callback to the meme that started it all. The recurring complaints are the same ones every good local model gets: it is slow, it eats VRAM, and the license is restrictive.

Performance numbers people are actually reporting:

- RTX 5090, 15-second 768p clip with one 13-second video reference plus 3 images: 57 minutes. Video references are the killer; the same generation with no references is far faster.
- 12 GB VRAM plus 24 GB system RAM works. Roughly 15 minutes for a 5-second clip, per a LocalLLaMA user.
- An unofficial Turbo LoRA (larryvrh/MiniMax-H3-Turbo-Lora) cuts generation to 6 to 10 steps, which is the difference between "fun experiment" and "usable in a workflow."

The MiniMax team did an AMA on r/StableDiffusion and confirmed the community's biggest gripes. Ref2VA output is visually weaker than FL2VA, and they said so directly, blaming divergent post-training strategies and promising fixes. They also confirmed the sparse-attention release is in the works and that the official 2K regeneration module will be open-sourced "once it is ready."

The other story of launch week: MiniMax issued takedowns on decensor and NSFW LoRAs. The Streisand effect was immediate, with mirrors and torrents popping up within hours. Worth a mention only because it tells you what the base model can already do; several users noted the base weights are "uncensored enough" for most purposes, so the LoRAs mostly existed to push further.

## The Catch: the License

H3 ships under the MiniMax H3 Community License Agreement, and it is not a permissive open-source license. The automatic grant covers "worldwide, excluding the Excluded Territories," and the Excluded Territories are the European Union, the United Kingdom, South Korea, and the United States. Commercial use, derivatives, and hosted services in those four regions require a separate application. One important carve-out: generated outputs are not treated as model derivatives, so the videos you create are yours to use even where the model itself is restricted.

That license is the main reason the "open weights" framing needs a footnote. Weights are open, the license is not OSI-approved, and the two most commercially active regions on earth need to ask permission. For individual tinkerers and anyone outside the excluded territories it is a non-issue. For agencies, studios, or SaaS products in North America or the EU, read the license before you build a business on it.

## How to Run It

Three paths, from easiest to most control:

1. **API:** platform.minimax.io, fal.ai (endpoints like `minimax/h3/text-to-video` and `minimax/h3/image-to-video`), OpenArt, and Runway all carry it. MiniMax claims the 2K per-second price is under a third of mainstream closed models, and 768p under half of mainstream 720p pricing.
2. **ComfyUI:** native T2V and R2V templates shipped within days of the release. This is the fastest way to a local workflow with LoRAs, reference nodes, and the rest of the ecosystem.
3. **Serve it yourself:** SGLang (the model card's reference deployment, 4 GPUs with Ulysses tensor parallelism) or vLLM for production, or the one-liner for a single GPU:

```python
from diffusers import DiffusionPipeline

pipe = DiffusionPipeline.from_pretrained("MiniMaxAI/MiniMax-H3", dtype="bfloat16", device_map="cuda")
video = pipe("Astronaut in a jungle, cold color palette, muted colors, detailed, 8k").images[0]
```

MiniMax also ships an agent skill for prompt writing, installable into Claude Code, Cursor, or any harness that reads SKILL.md files:

```bash
npx skills add https://github.com/MiniMax-AI/MiniMax-H3 --skill h3-prompt-writing
```

Their prompt guides matter more than usual here because H3 is CFG-distilled. There are no negative prompts, so prompt quality is the only lever you have. The prompt templates in the repo, which break a scene into shot-by-shot visual description, soundscape, and non-diegetic music blocks, are the single biggest quality multiplier available.

## Bottom Line

H3 is the strongest day-one open video release I have seen. The instruction following, the text rendering, and the native audio put it in a different class from the LTX and Wan releases that preceded it, and the community consensus agrees: people are deleting their old model folders. The audio-video coupling is the real step change. Sound is not synthesized to match a finished video; it is generated as part of the same latent sequence, which is why the boom lands on the cut and the score stops when the scene ends.

The caveats are real. The license excludes the US and EU without an application. Local inference is slow on consumer hardware, and reference-heavy prompts slow it further. The hosted pieces (Context-IR and 2K regeneration) mean the open model is only two-thirds of the advertised system. And Ref2VA needs quality work, which the team has acknowledged.

Still, the direction is unmistakable. One transformer, every modality, language as the interface. The next H-series generation promises M-series integration and higher resolution, the sparse-attention release should cut the quadratic attention cost that dominates long generations, and the technical report is on the way. If you build with video models, run the demos above and judge for yourself. Then go read the license before you commit to it.

## Sources

- MiniMax launch post: [MiniMax H3: An Open Model Breaking the Boundaries Between Tasks and Modalities](https://www.minimax.io/blog/minimax-h3)
- Model card: [MiniMaxAI/MiniMax-H3 on Hugging Face](https://huggingface.co/MiniMaxAI/MiniMax-H3)
- Code and prompt skill: [MiniMax-AI/MiniMax-H3 on GitHub](https://github.com/MiniMax-AI/MiniMax-H3)
- AMA: [MiniMax H3 Team, r/StableDiffusion](https://www.reddit.com/r/StableDiffusion/comments/1vh9rtw/ama_minimax_h3_team_ask_us_anything_about_our/)
- Launch threads: [r/StableDiffusion](https://www.reddit.com/r/StableDiffusion/comments/1ve4kue/minimax_h3_is_an_amazing_model/), [r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/comments/1ve1mvh/minimaxh3_now_on_huggingface/)
- Benchmarks and drama: [LTX 2.3 vs H3](https://www.reddit.com/r/StableDiffusion/comments/1vedlza/quick_comparison_ltx_23minimax_h3/), [Turbo LoRA](https://www.reddit.com/r/StableDiffusion/comments/1vgxf4x/minimax_h3_turbo_lora/), [LoRA takedowns](https://www.reddit.com/r/StableDiffusion/comments/1vfwijz/minimax_are_issuing_takedowns_on_decensorexplicit/)
- Hosted options: [fal.ai](https://fal.ai/minimax-h3), [Comfy](https://comfy.org/minimax-h3/), [OpenArt](https://openart.ai/ai-model/minimax-h3/)
- Coverage: [Forbes](https://www.forbes.com/sites/johnwerner/2026/08/03/4-things-to-know-about-minimax-h3/), [MarkTechPost](https://www.marktechpost.com/2026/08/01/minimax-releases-minimax-h3-an-omni-modal-video-model-that-generates-15-second-2k-clips-with-native-stereo-audio/)

*Demo clips are the official sample assets from the MiniMaxAI/MiniMax-H3 repository, self-hosted under /images/minimax-h3/.*
