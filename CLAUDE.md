# Agent Guide for Dvip Patel's Hugo Blog

This repository contains Dvip Patel's personal technical blog at `https://blogs.dvippatel.in/`. The site is built with Hugo and the PaperMod theme, with content focused on software engineering, AI systems, data products, and project write-ups.

## Project Snapshot

- Static site generator: Hugo
- Theme: PaperMod, installed as a Git submodule at `themes/PaperMod`
- Main config: `hugo.toml`
- Blog posts: `content/posts/*.md`
- Static assets: `static/`
- Generated site output: `public/`
- Privacy page: `content/privacy-policy.md`
- Custom head partial: `layouts/partials/extend_head.html`
- Cookie consent partial: `layouts/partials/cookie-consent.html`
- Production base URL: `https://blogs.dvippatel.in/`

Do not hand-edit files inside `public/`; regenerate them with Hugo.

## Common Commands

```powershell
# Preview locally, including drafts
hugo server -D

# Production build
hugo --gc --minify

# Create a new post
hugo new posts/my-post-slug.md
```

If the PaperMod theme is missing after cloning:

```powershell
git submodule update --init --recursive
```

## Content Conventions

Posts are Markdown files in `content/posts/`. Keep filenames descriptive and URL-safe, for example:

```text
content/posts/real-time-monitoring-ai-architecture.md
```

Use front matter that helps search engines, social previews, and answer engines understand the page:

```toml
+++
title = "Real-Time Monitoring AI Architecture"
date = 2026-05-11T23:22:00+05:30
draft = false
description = "How I designed a real-time object detection monitoring system with RTSP, MediaMTX, FastAPI, WebSockets, YOLO, and profiling."
summary = "A practical architecture write-up for a real-time AI monitoring system using RTSP streams, object detection, MediaMTX, FastAPI, WebSockets, and profiling."
tags = ["AI", "Computer Vision", "RTSP", "FastAPI", "YOLO"]
categories = ["Projects", "Architecture"]
keywords = ["Dvip Patel", "real-time monitoring AI architecture", "object detection monitoring system", "MediaMTX RTSP FastAPI"]
+++
```

Recommended writing pattern:

- Use one clear H1 from the post title and descriptive H2/H3 sections.
- Start with a short direct answer: what was built, who it is for, and what stack was used.
- Include Dvip Patel naturally in author/about context, not spammed repeatedly.
- Use descriptive image alt text.
- Link to relevant project repos, demos, docs, and related posts when available.
- Add tags and categories consistently.
- Prefer evergreen technical phrases that readers actually search for.

## SEO, Social, and Answer-Engine Priorities

The goal is discoverability across Google, Bing, DuckDuckGo, social previews, personal-site links, and AI/chatbot answer systems.

When editing the site, protect and improve:

- Stable canonical URLs under `https://blogs.dvippatel.in/`
- Descriptive page titles and meta descriptions
- Open Graph and Twitter card previews
- RSS feed availability
- Sitemap generation
- Clean internal linking between related posts
- Structured, factual, citation-friendly prose
- Author identity consistency: `Dvip Patel`, GitHub `dvip1`, X `PatelDvip`, LinkedIn profile from `hugo.toml`

For chatbot/AI discoverability, write pages that can be quoted and summarized easily:

- Include concise definitions and architecture summaries.
- Use lists/tables for systems, tools, and tradeoffs.
- Name technologies explicitly.
- Avoid burying the main point deep in a story-only intro.
- Keep dates accurate and visible.
- Link out to primary sources when referencing external tools or specs.

## Existing Customizations

`layouts/partials/extend_head.html` currently includes the cookie consent partial. Google Analytics script code is commented out there, while the GA4 ID is configured in `hugo.toml` under:

```toml
[services.googleAnalytics]
id = 'G-WLYGQMT86N'
```

Be careful when changing analytics or cookies; update `content/privacy-policy.md` if tracking behavior changes.

## Safe Editing Notes

- Keep edits scoped to content/config/layouts unless the task requires generated output.
- Do not overwrite user changes in content posts.
- Do not commit generated `public/` changes unless the project's deployment flow expects committed builds.
- After changing config, layouts, archetypes, or content, run `hugo --gc --minify` to verify the site builds.
- If adding images, place source assets under `static/images/...` and reference them with root-relative URLs like `/images/posts/post-slug/image.webp`.

