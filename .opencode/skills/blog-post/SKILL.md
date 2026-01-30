---
name: blog-post
description: Research, create, and deploy high-quality technical blog posts
license: MIT
compatibility: opencode
metadata:
  audience: users
  workflow: github
---

## What I do

- Orchestrate a high-quality technical blogging workflow for `blog.bymar.co`
- Conduct deep research using available MCP tools (web search, documentation, social context)
- Generate or source professional imagery for posts
- Create structured, informative content as "Marco" (founder of byMAR.CO)
- Build and deploy via Hugo and GitHub Actions

## When to use me

Use this skill when you need to:
- Create a new "Concept" blog post
- Publish technical insights or announcements
- Upgrade existing posts with better research or visuals

**CRITICAL CONTEXT**:
- The author is **Marco**, founder of bymar.co.
- The audience includes professionals in high-level positions.
- **Micro-management level**: High. Marco reviews EVERY post personally.
- **Quality Standard**: Extremely high. Content must be detailed, structured, and informative. No fluff.

## How to use use me

### Phase 1: Deep Research & Concept Validation

Before writing a single word of the post, you MUST conduct research to ensure the content is valuable and accurate.

1.  **Analyze the Request**: Identify the core technical concept.
2.  **Gather Context**: Use ANY available MCP tools to research:
    *   **Web Search**: Find the latest discussions, official docs, and similar articles.
    *   **Social Context (e.g. Reddit)**: Understand community sentiment and common pain points using available tools.
    *   **Documentation**: Use documentation search tools to get accurate API details/specs.
3.  **Synthesize**: meaningful, structured insights. Do not just summarize; provide value.

### Phase 2: Visual Asset Creation

Every post needs professional visuals.

1.  **Generate Images**: Use available image generation tools to create:
    *   A Hero image (abstract, technical, "premium" feel).
    *   Explanatory diagrams or charts if the concept is complex.
2.  **Source External Media**: If relevant valid images/videos are found during research, use `curl` to download them (respecting usage rights).
3.  **Storage**:
    *   Create a directory: `static/images/posts/[post-slug]/`
    *   Save all assets there.
    *   **Verify**: Ensure files exist before referencing them.

### Phase 3: Content Creation

Create the content in `content/posts/[post-slug].md`.

**Frontmatter Template**:
```yaml
---
title: "Title Here"
date: 2026-01-30T12:00:00-05:00 # USE CURRENT TIME with timezone
draft: false
description: "Compelling, technical description for SEO."
summary: "Executive summary of the concept."
tags: ["tech", "ai", "hugo"] # Relevant tags
categories: ["concepts"]
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
  image: "" # Leave empty if no hero image
  alt: ""
  caption: ""
  relative: false
  hidden: true # Hidden by default
---
```

**Content Guidelines**:
- **Voice**: Authoritative yet accessible. "I" refers to Marco.
- **Structure**: Use clear H2/H3 headers. Use bullet points for readability.
- **Evidence**: Cite sources from your research. Link to official docs.
- **Header Requirements**: Must include ALL standard Hugo frontmatter fields shown in template above, matching the style of existing posts.

### Phase 4: Build & Deploy

1.  **Build**:
    ```bash
    hugo --minify
    ```
    *Fix any build errors immediately.*

2.  **Publish**:
    Push directly to the `main` branch to trigger the production deployment.
    ```bash
    git add content/posts/[post-slug].md
    git commit -m "feat(blog): publish concept [post-title]"
    git push origin main
    ```
    *Note: Only commit the markdown file, not images (unless they exist in static/)*

### Phase 5: Final Output

Once pushed, provide the direct link for Marco's review:

> **Concept Published for Review**
> URL: https://blog.bymar.co/posts/[post-slug]/
>
> Please review the content, accurate sourcing, and generated visuals.
