# Khushal / Enterprise Architecture & Systems

Astro portfolio, Markdown blog, and browser-based presence simulation. Hosted at https://khushal075.github.io/.

## Write a post

Create `src/content/blog/your-post.md`:

```markdown
---
title: "Your article title"
description: "A concise summary."
date: 2026-10-29
tags: ["Enterprise architecture"]
draft: true
---

Write your article here.
```

Set `draft: false` when ready to publish. Drafts are excluded from article pages, the archive, and RSS, but their source remains visible in this public repository. Commit and push to `main`; GitHub Actions builds and deploys the site. Article URLs use the file name: `/blog/your-post/`.

## Develop

Node.js 24 or later:

```bash
npm ci
npm run dev
npm run build
npm run preview
```

Site copy is in `src/components/home.html`; shared styles are in `src/styles/global.css`. The simulation is in `public/playground.js`. Blog routes and the RSS feed are in `src/pages/`. The deployment workflow is `.github/workflows/deploy.yml`.

The playground uses simulated data. There is no AI backend or live production telemetry.
