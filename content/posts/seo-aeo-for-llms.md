---
title: "SEO is still alive. AEO is the new surface area (SEO for LLMs)."
date: 2026-03-02T19:30:00-05:00
draft: false
description: "A practical playbook for getting cited by ChatGPT, Perplexity, and Google AI Overviews. Structure, crawlability, authority, and measurement."
summary: "If you want to show up in LLM answers, you need to be easy to retrieve and easy to quote. This post lays out the new mechanics, the 80/20 tactics, and how to measure what’s working."
tags:
  - seo
  - aeo
  - llm
  - ai-overviews
  - marketing
  - web
categories:
  - growth
author: "Marco"
showToc: true
TocOpen: false
comments: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
cover:
  hidden: true
---

Traffic is splintering.

Classic SEO still matters, but a growing chunk of “search” ends in an AI answer, not a blue link. So the optimization target shifts from **ranking** to **being the source the model cites**.

People call it AEO, GEO, LLMO, whatever. The name doesn’t matter. The mechanics do.

## The new game: citations, not clicks

Google AI Overviews and “LLM search” products do 3 things:

1) Retrieve candidate sources  
2) Decide which sources are “safe to trust”  
3) Synthesize an answer (sometimes with citations)

If you want to win, your content needs to be:

- **Retrievable** (crawlable, indexable, present where retrievers look)
- **Extractable** (easy to quote in a tight, factual chunk)
- **Credible** (the web agrees you’re a real authority)
- **Measurable** (you can tell if you’re showing up)

{{< linkcard
  url="https://x.com/marc_laclear/status/2028438289422659967"
  title="AI Overviews citation dynamics: only 38% of citations from top SERP results (down from 76%)"
  site="X"
  author="@m_arc_laclear"
>}}

### SEO vs AEO: the overlap is not 100%

A useful mental model is: **AEO = SEO + citation readiness + web consensus.**

Also, different engines pull from different corpuses. The overlap between Google results and what ChatGPT cites can be surprisingly low.

{{< linkcard
  url="https://x.com/apoorvshrm/status/2027108311871738062"
  title="Google AI Overviews vs ChatGPT citations: reported 8% URL citation overlap"
  site="X"
  author="@apoorvshrm"
>}}

Takeaway: if your entire content strategy assumes “rank in Google = appear in LLM answers,” you’re betting the channel on an assumption that’s already failing.

## The AEO playbook (the 80/20 that keeps showing up)

### 1) Write answer-first pages

The easiest thing for a model to quote is a clean, early answer.

Do this:

- Put the **direct answer in the first 2 to 3 sentences**
- Use **question headings** that match how people prompt (“What is X?”, “How do I do Y?”)
- Add **tight bulleted lists** and **tables** (models love chunkable structure)
- Add a short **FAQ section** (real questions, not keyword soup)

Avoid:

- 800 words of “context” before you define the thing
- Vibes-only writing that can’t be cited (“we believe…”, “it’s changing everything…”)
- Giant walls of text with no structure

### 2) Build topic clusters, not one-off posts

LLMs (and modern retrieval) reward coverage.

You want:

- 1 pillar page (the hub)
- 5 to 20 supporting pages that answer sub-questions
- internal links that make the relationships obvious

This is classic SEO, but with an AEO twist: your supporting pages should each have **a quotable answer block**.

### 3) Make your site readable to machines (seriously)

A depressing number of modern sites are “human readable” but “LLM hostile.”

A strong heuristic: minimize the ratio of navigation/scripts to actual content. Make the main content obvious in the raw HTML. Server-render where possible. Clean headings. Clean anchors.

{{< linkcard
  url="https://x.com/itsJaimeMedina/status/2028532407599866352"
  title="Many sites are effectively invisible to AI agents because HTML is dominated by nav/scripts"
  site="X"
  author="@itsJaimeMedina"
>}}

Practical checklist:

- Don’t block relevant bots in `robots.txt` (and don’t block your own content behind JS-only rendering)
- Ship a real `sitemap.xml` with accurate `lastmod`
- Ensure canonicals are correct
- Make “main content” easy to parse (semantic HTML, not div soup)

### 4) Win web consensus, not just backlinks

For LLM visibility, the strongest pattern isn’t a single backlink. It’s **repeated, consistent mentions across trusted places**.

Examples:

- “Best X tools” listicles
- Comparison pages (“X vs Y”)
- Review sites / directories in your category
- Dev docs, GitHub, community threads (when relevant)

Think of this as **category association**. When the model sees the same brand repeatedly in the same role, it becomes the default answer.

### 5) Schema is table stakes, not a strategy

Schema can help with eligibility and extraction, but it’s not the lever people think it is.

I agree with this framing: lots of “AEO” work is just repackaged content + schema, with zero measurement.

{{< linkcard
  url="https://x.com/bmadigan/status/2028557329650585894"
  title="Most AEO work is content + schema + FAQs, without measuring whether AI systems surface the brand"
  site="X"
  author="@bmadigan"
>}}

Use schema, but don’t pretend it’s the whole playbook:

- `FAQPage` (when real)
- `HowTo` (when real)
- `Product`, `SoftwareApplication`, `Organization`, `Article`
- clean author info and publish dates

## Measurement: if you can’t measure it, you can’t iterate

AEO measurement is annoying because “rank tracking” doesn’t map cleanly.

What does work:

### 1) Prompt-based tracking (cheap, directional)

Pick 20 to 50 prompts you care about. Run them weekly across:

- ChatGPT
- Perplexity
- Gemini / AI Overviews (where possible)

Track:

- Are you mentioned?
- Are you cited/linked?
- Which page is cited?
- Which competitors show up instead?

### 2) Log-based tracking (more real)

Watch your server logs for known bots and referrers, and tag them in analytics.

Also, monitor “AI referral” sources (Perplexity and some products do refer). This is early and inconsistent, but it’s signal.

### 3) Content experiment loops

AEO is an extraction problem. Run page-level experiments:

- add an answer block
- add a comparison table
- tighten headings into question form
- add 5 internal links from related pages

Then re-check citations 7 to 14 days later.

## What I’d do for byMAR.CO specifically

If I were optimizing bymar.co and blog.bymar.co for LLM visibility, I’d ship:

1) 3 pillar pages (definition hubs)  
2) 15 supporting answer pages (1 question each)  
3) 5 comparison pages (people prompt these constantly)  
4) an About / credibility pass (author pages, experience, links, clear identity)  
5) a measurement sheet (prompt set + weekly checks)

If you tell me the target category (agents? OpenClaw? AI automation? local models?), I’ll propose the exact pillar/supporting map and the first 10 posts.

## A quick note on Reddit + real users

People are increasingly annoyed by AI layers, and they still click through to sources when they care. That’s good news if your content is actually useful.

{{< linkcard
  url="https://www.reddit.com/r/digital_marketing/comments/1reddq9/do_you_trust_ai_overview_responses_alone/"
  title="Reddit: Do you trust AI Overview responses alone? (many users still click sources)"
  site="Reddit"
  author="r/digital_marketing"
>}}

Bottom line: AEO isn’t magic. It’s making your content easy to retrieve, easy to quote, and hard to ignore because the web repeatedly confirms you’re a legitimate source.

If you want this channel, build pages that models can cite without thinking.
