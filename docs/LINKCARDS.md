# LINKCARDS (byMAR.CO Hugo)

## Why
Raw Reddit/X URLs look dead. We use a minimal “link card” shortcode so important sources are visually scannable and clickable.

This avoids:
- flaky OG scraping at build time
- embed JS (tracking + perf hit)

## Files
- Shortcode: `layouts/shortcodes/linkcard.html`
- CSS: `static/css/linkcard.css`
- Included globally via: `layouts/partials/extend_head.html`

## Usage
```md
{{< linkcard
  url="https://x.com/LangChain/status/2019846811997942219"
  title="LangChain: Debug voice agents with LangSmith tracing (Pipecat)"
  site="X"
  author="@LangChain"
>}}
```

Optional thumbnail (preferred if we have a good one):
```md
{{< linkcard
  url="https://www.reddit.com/r/openclaw/comments/.../"
  title="Does OpenClaw actually do anything for you guys?"
  site="Reddit"
  author="u/ElmangougEssadik"
  image="/images/linkcards/openclaw-thread.jpg"
>}}
```

## Conventions
- X: `author="@handle"`
- Reddit: `author="u/username"`

## Policy (moving forward)
- In bymarco-blog posts, prefer `linkcard` for **X** and **Reddit** references.
- Keep normal markdown links for “boring” references (docs, repos) unless it’s a primary source you want clicked.
