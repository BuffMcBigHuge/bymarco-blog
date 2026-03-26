# bymarco-blog posting notes

These are repo-specific rules for publishing new posts to `development/bymarco-blog`.

## Frontmatter rules

For normal posts under `content/posts/*.md`:

- **Prefer `slug` and do not set `url` manually** unless there is a deliberate migration / alias reason.
- This repo has existing posts with explicit `url`, but for new posts the safe default is:
  - `title`
  - `date`
  - `draft: false`
  - optional `slug`
  - `description`
  - `summary`
  - tags/categories/etc.
- If you need legacy URL preservation, use aliases deliberately and verify the generated route.

### Safe default frontmatter template

```yaml
---
title: "Post title"
date: 2026-03-26T12:00:00-04:00
draft: false
slug: "post-slug"
description: "One-sentence description."
summary: "Short summary."
cover:
  image: "/images/posts/<slug>/hero.png"
  alt: "Describe the image"
  caption: "Optional caption"
  relative: false
  hidden: false
tags:
  - ai
categories:
  - ai-tools
author: "Marco"
showToc: true
TocOpen: false
comments: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
---
```

## Image rules

When publishing a serious post, add at least:

1. a cover image in `static/images/posts/<slug>/`
2. ideally 1-2 inline images/diagrams if the topic is conceptual or systems-heavy

Keep image paths absolute from the site root, e.g.:

```md
![Alt text](/images/posts/<slug>/hero.png)
```

## Link/source rules

- For X and Reddit references in bymarco-blog posts, prefer the `linkcard` shortcode when appropriate.
- Keep raw source dumps out of the body. Synthesize them.

## Publish verification checklist

Do **not** say a post is published just because `git push` succeeded.

After pushing:

1. Confirm the commit is on `origin/main`
2. Check the production workflow run for the pushed SHA
3. Confirm the new route exists in build/deploy evidence, or verify on the live site:
   - homepage/post listing includes the new post, or
   - direct post URL resolves to the article, not homepage fallback
4. If live verification fails, inspect frontmatter/routing before blaming deploy infrastructure

## Known footgun

### Explicit `url` on new posts

A new post was pushed with:

```yaml
url: "/posts/agent-first-tools-are-a-real-category/"
```

In this repo/setup, that resulted in the production build completing while the expected post route was not emitted as a live article page. The site fell back to homepage-like behavior for the route.

**Rule going forward:** for new posts, do not set `url` manually unless you have a specific reason and have verified the route generation behavior.
