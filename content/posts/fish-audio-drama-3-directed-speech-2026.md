---
title: "Fish Audio Drama 3: The First TTS Model You Direct Instead of Prompt"
date: 2026-10-26T21:00:00-04:00
draft: false
description: "Drama 3 is a 35B open-weights speech model that takes stage directions and runs in ComfyUI on an 8 GB card. Specs, syntax, and the TTS race."
summary: "Fish Audio's Drama 3 is a 35B MoE that acts instead of reading: free-text direction in brackets, word-scoped tags, 16 speakers, and retakes you can revise by note. How it works, how to run it locally in ComfyUI on 8 GB, and what the open-weight TTS race is doing to the market."
tags:
  - ai
  - text-to-speech
  - audio
  - open-weights
  - comfyui
  - voice-cloning
  - local-llm
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
  image: "images/drama3-directed-speech/overview.png"
  alt: "Dark editorial diagram of a screenplay page feeding through a 35B MoE chip into an audio waveform, labelled Fish Audio Drama 3"
  caption: "Directions in the script, audio out. Drama 3's whole thesis in one diagram."
  relative: true
  hidden: false
---

Every text-to-speech model in 2025 was a reader. You handed it a sentence, it handed back a sentence-shaped sound, and if the delivery was wrong you rewrote the prompt or rolled a new seed. The performance was a side effect of the text.

Fish Audio Drama 3 treats the performance as the input. You write stage directions inline, in plain language, wherever you want the delivery to shift. You scope a tag to three words instead of a whole line. You send a note about the last take and the model revises that take instead of starting over. Then you find out it's 35 billion parameters, the weights are on Hugging Face, and the community has it running in ComfyUI on an 8 GB card.

I read the [technical report](https://arxiv.org/abs/2610.18010), the [model card](https://huggingface.co/fishaudio/drama-3), the ComfyUI partner docs, and the [local node pack](https://github.com/PedroMarinhoDev/ComfyUI-Drama3), then went through the launch threads on r/LocalLLaMA and X. Here's what's actually new, what it costs to run, and why the competition around it matters more than the leaderboard position.

## What Drama 3 is

A 35B-parameter mixture of experts from Fish Audio, 3B active per token. It's a Qwen3.5-style hybrid backbone: 40 layers, attention plus DeltaNet, 256 routed experts per layer of which 8 fire per frame, a shared expert, and a 13-layer codebook decoder that generates 10 codebooks causally across the full sequence.

That decoder is the real architectural change. Older Fish models predicted audio frames token by token. Drama 3 predicts each frame's codebooks in dedicated layers that attend over the whole sequence, which is why the bigger model is also the faster one: 271 frames per second on one H200 against 177 for S2, with seven times the parameters.

What you can actually do with it:

- **16 speakers in one script**, marked `<|speaker:0|>` through `<|speaker:15|>`, each cast from a reference clip
- **Free-text direction** instead of a fixed emotion taxonomy. If you can brief an actor, you can write the tag.
- **Word-scoped tags** that change only the words they wrap
- **Multi-turn revision.** Send your note with the earlier takes in context, and it redoes the line
- **Infilling edits.** Replace one word of existing speech and keep the rest
- **83 languages**, per Fish. The model card front matter lists 79.

Hugging Face shows 23,216 downloads and 1,037 likes, which understates real traffic since most people pull community quants.

## Why it can act: the reward model was the problem

This is the part worth understanding, because it explains why every other open TTS model sounds like a very good audiobook narrator and nothing more.

Standard TTS reinforcement learning optimizes two proxies: word error rate from an ASR model, and speaker similarity from a timbre model. Both are reasonable. Both actively punish acting.

The report measured it on Expresso pairs, where two human readings of a sentence differ only in style, against a default-style clip as the similarity reference:

- Whispering drops similarity by 0.310. A similarity reward prefers the flat default reading in **99.7%** of those pairs.
- Reading with a laugh raises WER by 0.79 points, enough that a WER reward penalizes it. Similarity prefers the flat reading in **93.9%** of laughing pairs.
- Even "clearly enunciated reading" loses to default reading in 62.0% of pairs.

So a model trained to maximize those two numbers learns that the safest audible output is calm, even, and boring. That's exactly what open TTS sounded like for two years.

Drama 3's post-training uses a single learned model of human preference as the only reward, and the same pairs flip. With a direction present, the reward model prefers the whispered take in 62.2% of pairs and the laughing take in 64.4%, against 10.6% and 27.1% without the direction. The direction is doing work in the objective, not just in the prompt.

That's the paper's contribution. The architecture is good engineering. The training signal is the insight.

## Directing it

Two syntaxes exist, and this trips people up.

In the raw model and the local node pack, speaker turns use `<|speaker:0|>` and scoped tags use angle brackets:

```text
<|speaker:0|> [icy, already decided] So this is goodbye.
<|speaker:1|> [small laugh, not amused] Promise me <whisper>you won't tell her</whisper>.
<|speaker:0|> Fine. [sighs] Go.
```

In the hosted ComfyUI partner nodes you connect voices to the node's `voices` slots and mark turns with `@Voice1`, `@Voice2`. Every connected voice must appear in the text at least once or the node errors:

```text
@Voice1 [deep, cinematic narrator] Every night, the city tells a story.
@Voice2 [soft, close, like a secret] But only one voice ever finishes it.
@Voice1 [drops to a whisper, then brightens] Fish Audio Drama 3, <whisper>now in ComfyUI</whisper>.
```

The scoped tags are `<whisper>`, `<emphasis>`, `<soft>`, `<fast>`, `<slow>`, `<stress>`, `<forceful>`. Point tags like `[sighs]`, `[laughs]`, `[gasps]`, `[pause]`, `[clears throat]` land wherever you put them. There's also a formal script layout with `Direction:` on one line and `Line:` on the next, and a `[language: en]` hint that isn't spoken.

Two rules are worth knowing. A direction holds until the next one, so `[calm]` at the top of a line governs the whole line. And directions don't have to match the script's language: an English line directed `[像是在强忍着眼泪]` works.

The feature I'd actually pay for is multi-turn revision. Every other model makes you re-roll: change a word, lose the timing you liked, re-roll again. Drama 3 keeps the take and the note in context.

### Hear it directed

One take, straight out of ComfyUI, nothing edited. This is the exact script that went in:

```text
[calm, close, like the room is empty] Nine out of ten voice models read the line. [a beat, almost amused] This one waits for the direction. [drops to a half-whisper] And that's where <emphasis>everything</emphasis> changes.
```

{{< audio-sample title="Drama 3: three directions in one take" srcMp3="/images/audio/drama3-directed-speech/direction-shift.mp3" caption="7.9 seconds, one generation. The delivery shifts mid-take and the last line drops to a half-whisper, no re-roll." meta="Measured on the file: the first phrase sits at -25 dB RMS, the last at -34 dB. That 9 dB drop is the direction working, and it is why the scoped tag exists." notes="Loudness normalized to -17.5 LUFS with a -1.5 dBTP limiter for the web. No compression, EQ, or editing." >}}

Listen for the beat at 0:02, where `[a beat, almost amused]` inserts about 1.3 seconds of nothing. That's the part most open TTS models refuse to do. They close the gap because a pause looks like a defect to a word-error metric, which is the whole argument from the previous section, audible in eight seconds.

## Running it locally in ComfyUI

You have two paths.

**Hosted partner nodes.** ComfyUI ships four Fish Audio nodes under `partner/audio/Fish Audio`: Text to Speech, Speech to Text, Voice Selector, Instant Voice Clone. Set `model` to `drama-3`, connect up to 5 voices, and the two Drama 3 templates (Directed Performance, Two-Speaker Dialogue) need ComfyUI 0.39.3 or newer. Pricing is about $14.30 per 1M bytes of text during the launch offer, then **$35.75 from November 13, 2026**. Voice cloning is free, speech to text is $0.00858 per minute.

**Local weights.** This is what makes Drama 3 a story instead of a hosted demo. [ComfyUI-Drama3](https://github.com/PedroMarinhoDev/ComfyUI-Drama3) is a native node pack, GPL-3.0, 110 stars, created October 14. No `pip install`, no extra dependencies:

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/PedroMarinhoDev/ComfyUI-Drama3
```

Then pull three files:

| File | Folder | Size |
|---|---|---|
| `drama3_int8experts_bf16.safetensors` | `models/diffusion_models` | 38.7 GB |
| `drama3_codec.safetensors` | `models/vae` | 1.6 GB |
| `qwen3_asr_0.6b_bf16.safetensors` | `models/asr` | 1.6 GB |

```bash
hf download PedroMarinhoDev/Drama-3-ComfyUI drama3_int8experts_bf16.safetensors --local-dir ComfyUI/models/diffusion_models
hf download PedroMarinhoDev/Drama-3-ComfyUI drama3_codec.safetensors --local-dir ComfyUI/models/vae
hf download PedroMarinhoDev/Qwen3-ASR-ComfyUI qwen3_asr_0.6b_bf16.safetensors --local-dir ComfyUI/models/asr
```

The trick is the memory split. About 6 GB of the model isn't routed experts and stays on the GPU. The 32 GB of experts stay in system RAM, and the free VRAM becomes a cache: at 15 GB, 83% of expert reads hit. The pack's own CUDA kernels, compiled at runtime by NVRTC, read the missed experts straight from RAM over PCIe while the CPU computes others, and every layer replays as a CUDA graph.

![Dark technical diagram showing GPU VRAM holding attention, DeltaNet, shared expert, codebook decoder and an expert cache, connected over PCIe to a 32 GB grid of experts in system RAM](/images/drama3-directed-speech/routing.png)

*The split that puts a 35B model on an 8 GB card. Experts live in pinned system RAM; the GPU caches the hot ones.*

Requirements are steeper than the VRAM number suggests: an NVIDIA GPU from 8 GB, but **32 to 48 GB of system RAM** and 26 to 39 GB of disk. Measured on a 9-second line after the October 17 kernel update:

| GPU | File | Expert cache | 9 s line | Peak VRAM |
|---|---|---|---|---|
| RTX 4090 | W4A8 | 16 GB | 3.9 s | 21.8 GB |
| RTX 4090 | int8 | 16 GB | 6.1 s | 21.8 GB |
| RTX 3090 | int8 | 16 GB | 6.7 s | 21.8 GB |
| RTX 3090 (held to 8 GB) | int8 | automatic | 9.9 s | 6.7 GB |
| RTX 3090 + 4090 | int8 | experts in VRAM | 2.8 s | 37.5 GB |

Real time is 46 ms per frame at 21.5 frames per second of speech. One 3090 runs about 1.3× faster than real time. An 8 GB slice is roughly at parity. Two cards get you close to 3×.

On file choice, start with int8. FP8 holds Fish's exact expert weights but falls apart on small caches, 20.7 s for that same 9 seconds at 8 GB. W4A8 is the smallest at 24.6 GB, and the pack's own testing found it "falls silent more often with real voice clips." int8 on a 24 GB card is the accurate and the fast option at the same time.

The node set goes past a generate button: encode and save a voice as a few-KB `.safetensors`, reload it without the codec, save takes so you can re-decode without regenerating, and an Editor with word-level timing from Qwen3-ForcedAligner.

### The failure mode you'll hit first

Silent takes. Forty seconds of work, no audio.

The cause is documented and specific. The pack inherited `top_k 30` from Fish's server, and Fish's own test found that left **11 of 40 H200 takes silent at pauses**. Set `top_k` to 0, keep temperature at 1.0, top_p at 0.9. Workflows saved before the fix keep the old value, so check it.

Second cause: the reference transcript has to cover every word of the clip. Cut the clip between sentences, or Qwen3-ASR writes a transcript that misses the last words and the take goes quiet.

Third, dialogues. If a speaker has no assigned voice, chunked generation hands the previous chunk's voice to the next one. The pack measured 6 of 16 lines coming out in the wrong voice with two cloned voices; with every speaker voiced, 1 of 18.

## The competition, and why it's the good news

Here's the honest state of the leaderboard, from Artificial Analysis today.

On the provider voice board, where each model uses its own voices, Drama 3 Preview sits at **#10 with an Elo of 1245** at $15 per 1M characters. Eleven v4 and v4 Turbo tie at the top on 1333, at $80 and $40 per 1M characters respectively.

On the controlled voice board, where every model clones the same 8 voices, Drama 3 Preview is **#17 at 1061**. That's 100 Elo behind Eleven v4 at 1161. It's also behind Ming-3.0-tts at 1072, Ant Group's dense 0.5B model under an MIT license, and just ahead of Breeze TTS 2 at 1019 and Voxtral TTS at 1005.

Fish's "5x cheaper" line is true against Eleven v4 at $80 per 1M characters. Against v4 Turbo at $40, it's 2.7×. Pick your comparison.

Two things follow, and they point in opposite directions.

**The lead is real but narrow.** Drama 3 launched October 13 and was the top open-weights TTS model on Artificial Analysis for about a week. Ming-3.0-tts took that spot on October 20, on a model 70 times smaller with an MIT license. If you're choosing a model today, the argument for Drama 3 isn't the leaderboard. It's the direction syntax, the 16-speaker scenes, and the revision loop, none of which a competitor matches yet.

**The race is doing the work that matters.** Inside two weeks there were four independent runtimes Fish never wrote:

- [audio.cpp](https://github.com/0xShug0/audio.cpp) merged Drama 3 with Q4/Q4/Q2 GGUF quants, reporting up to 5× real time at full speed and 2× real time under 8 GB of VRAM on an RTX 5090
- CTranslate2 4.8.4 added Drama 3 inference on CPU and GPU, which puts a 35B TTS model on hardware with no GPU at all
- MLX and TensorFold builds put it on Apple Silicon, including a 20 GB quant
- `drama3-small` pruned, quantized, and distilled the 35B model to about 9.7 GB, running on a 12 GB RTX 3060

Jmica's GGUF repo alone has 311,675 downloads, thirteen times the official checkpoint. That's the pattern that matters: the weights are the starting gun, not the product.

The competition is also what makes the local option viable at all. A year ago, open-weights TTS meant Kokoro 82M, fast and thin, or a fine-tune of somebody's research repo. Today you can run a directable 35B actor on a 3090 faster than real time, a 0.5B MIT-licensed model ties it on pronunciation, and the whole ladder is public. Voice is turning into a commodity input. Knowing how to direct one isn't.

## What the community found

The r/LocalLLaMA launch thread hit 452 upvotes and 94 comments, a lot of attention for a TTS model this size. Reception split cleanly.

The enthusiasm is about capability. "Best I've seen so far." "It supports up to 16 speakers in the same transcript... but also revision instructions 'a bit sadder' to incrementally get the voice line where you want it." The H100 streaming numbers, a 0.32 real-time factor, took the top comment and the obvious reply: your local H100.

The complaints are specific and worth hearing.

**Clone fidelity is the weak spot.** "It's more like a cousin of the reference voice rather than..." Fair. Drama 3 is trained for performance, not forensic identity match. If you need a likeness, Qwen3-TTS fine-tunes and Omnivoice do better, and the same thread said so.

**Some outputs still sound dead.** "From the samples I heard it still sounds a bit dead to me. Breeze TTS 2 is probably the only open weight model that sounds alive." That one suggests the direction syntax only pays off when you use it. A flat reference clip produces a flat take regardless, as the node pack's docs say outright.

**The license annoys people.** One commenter called it "a soft restrictive licence for commercial use," and the reply under it was blunter: "you can play around freely, but for commercial use it's hardly king."

One detail from r/comfyui that no benchmark shows. The pack's author measured int8 at the same frame time on a 3090 and a 4090 before the October 17 kernels, because decoding one frame at a time leaves a single core launching thousands of small GPU kernels. Your CPU matters more than your GPU tier. That's also why the 8 GB rows above hold up on int8 and collapse on FP8.

## The license, plainly

Drama 3's weights are under the Fish Audio Research License, not an OSI license. Research and non-commercial use is free. Commercial use is free below **USD 500,000 annual revenue**. Above that, or if you want to run a hosted TTS service on it, you need a license from Fish. Cloning an identifiable person without consent is out.

That's a normal research-lab compromise and a real constraint. Under the threshold it changes nothing. Build a product that could cross it, and you're building on rented ground. Put that next to Ming-3.0-tts under MIT, and the difference in legal risk is bigger than the difference in Elo.

Fish isn't a tiny lab either: a $52M seed round in July, and 15 million users claimed at Drama 3's launch. They have every reason to keep the weights open as a funnel to the API, and to keep the revenue threshold as the commercial gate.

## Bottom Line

Drama 3 is the first open TTS model where the interesting part is the interface. Directions that read like notes to an actor, tags that stop at the edge of a word, takes you can revise instead of re-roll, 16 voices in one script.

The engineering is clever too. Putting 90% of a 35B model in system RAM and reading the misses over PCIe, with CUDA graphs per layer, is what turns a data-center checkpoint into a tool a 3090 owner can use. Eight gigabytes of VRAM and 48 of RAM gets you a take.

I wouldn't bet the architecture on this checkpoint, though. The open-weight crown changed hands twice in two weeks and the license has a revenue ceiling. Build on the directed-script format and the revision loop, which are portable. Treat the weights as replaceable.

If you write audio for games, audio drama, or video, this is the best free tool you have for getting a performance instead of a read. If you need a forensic voice clone, it isn't, and the commenters on the download page will tell you.

Run the two Comfy templates first. Then install the local pack and set `top_k` to 0 before you decide the model is broken.

## Sources

- Technical report: [Fish Audio Drama 3 Technical Report, arXiv 2610.18010](https://arxiv.org/abs/2610.18010)
- Model card and weights: [fishaudio/drama-3 on Hugging Face](https://huggingface.co/fishaudio/drama-3)
- ComfyUI docs: [Fish Audio Drama 3](https://docs.comfy.org/tutorials/partner-nodes/fishaudio/drama-3), [Fish Audio nodes](https://docs.comfy.org/tutorials/partner-nodes/fishaudio/fish-audio), [prompt guide](https://docs.comfy.org/tutorials/partner-nodes/fishaudio/prompt-guide)
- Local node pack: [PedroMarinhoDev/ComfyUI-Drama3](https://github.com/PedroMarinhoDev/ComfyUI-Drama3), weights at [Drama-3-ComfyUI](https://huggingface.co/PedroMarinhoDev/Drama-3-ComfyUI)
- Leaderboards: [Artificial Analysis provider voice](https://artificialanalysis.ai/text-to-speech/leaderboard), [controlled voice](https://artificialanalysis.ai/text-to-speech/leaderboard/controlled-voice)
- Community threads: [New TTS king in town: Fish's Drama 3 35B3A](https://www.reddit.com/r/LocalLLaMA/comments/1x6f9p2/new_tts_king_in_town_fishs_drama_3_35b3a/), [custom nodes for ComfyUI at 8 GB](https://www.reddit.com/r/comfyui/comments/1x5ynn4/), [Drama3 in audio.cpp](https://www.reddit.com/r/LocalLLaMA/comments/1x75uyi/)
- Ports and quants: [audio.cpp](https://github.com/0xShug0/audio.cpp), [Jmica/Drama-3-GGUF](https://huggingface.co/Jmica/Drama-3-GGUF), [mlx-community 4bit](https://huggingface.co/mlx-community/fish-audio-drama-3-4bit), [pentacoxian-dev/drama3-small-GGUF](https://huggingface.co/pentacoxian-dev/drama3-small-GGUF)
- X: [@FishAudio on the reward change](https://x.com/FishAudio/status/2112597163922641058), [@ArtificialAnlys on the open-weights ranking](https://x.com/ArtificialAnlys), [@ichelpark on directing dialogue](https://x.com/ichelpark/status/2111106737386484090), [@jordimash on CTranslate2 4.8.4](https://x.com/jordimash/status/2114437710978932984)
