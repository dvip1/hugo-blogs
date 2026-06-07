# Dvip Patel Blog

Personal technical blog for Dvip Patel, published at `https://blogs.dvippatel.in/`. The site is built with Hugo and the PaperMod theme, and currently focuses on engineering notes, AI systems, secure data products, and project architecture write-ups.

## Tech Stack

- Hugo static site generator
- PaperMod Hugo theme as a Git submodule
- Markdown content in `content/`
- Static assets in `static/`
- Generated output in `public/`
- Google Analytics 4 configured in `hugo.toml`
- Hosted on AWS Amplify, according to the privacy policy

## Repository Layout

```text
.
|-- archetypes/              # Templates for new Hugo content
|-- content/
|   |-- posts/               # Blog posts
|   `-- privacy-policy.md
|-- layouts/
|   `-- partials/            # Local PaperMod overrides/custom head content
|-- static/                  # Images and other public static files
|-- themes/
|   `-- PaperMod/            # Theme submodule
|-- public/                  # Generated Hugo build output
|-- hugo.toml                # Site configuration
`-- CLAUDE.md                # Agent-facing project guide
```

Do not edit `public/` by hand. Change `content/`, `layouts/`, `static/`, or `hugo.toml`, then rebuild.

## Getting Started

Clone the repository with submodules:

```powershell
git clone --recurse-submodules <repo-url>
cd hugo-blogs
```

If the repository was already cloned without submodules:

```powershell
git submodule update --init --recursive
```

Install Hugo Extended if it is not already available:

```powershell
hugo version
```

Run the local development server:

```powershell
hugo server -D
```

Create a production build:

```powershell
hugo --gc --minify
```

## Creating Posts

Create a new post:

```powershell
hugo new posts/my-post-slug.md
```

Recommended post front matter:

```toml
+++
title = "Post Title"
date = 2026-06-07T14:30:00+05:30
draft = true
description = "A one-sentence search and social preview for this post."
summary = "A short summary of what the post explains and why it matters."
tags = ["AI", "Architecture", "FastAPI"]
categories = ["Projects"]
keywords = ["Dvip Patel", "specific topic people search for"]
+++
```

For strong search and social previews, every finished post should have:

- A clear title that includes the real topic.
- A `description` under roughly 160 characters.
- Useful `tags`, `categories`, and `keywords`.
- Descriptive image alt text.
- Links to related posts, source code, demos, or primary docs where relevant.
- A concise opening paragraph that directly says what the post is about.

## SEO Goals

The blog should be easy to discover when people search for:

- `Dvip Patel`
- Dvip Patel's project names and technical write-ups
- Topics covered in posts, such as real-time object detection, RTSP monitoring architecture, secure text-to-SQL analytics, FastAPI, MediaMTX, YOLO, and AI dashboards

The site should also produce good link previews on social sites and personal websites, and be easy for answer engines and chatbots to summarize accurately.

Important SEO habits:

- Keep `baseURL` in `hugo.toml` set to the production domain.
- Keep URLs stable after publishing.
- Use descriptive front matter on every post.
- Publish an RSS feed and sitemap through Hugo.
- Add internal links between related posts.
- Add external links to authoritative sources when explaining tools or specs.
- Use specific headings that match likely searches.
- Include Dvip Patel's author identity consistently across the site, profiles, and structured metadata.

## Current Site Identity

Configured in `hugo.toml`:

- Site title: `Dvip Patel`
- Author: `Dvip Patel`
- Home intro: personal blog by Dvip Patel
- X: `https://x.com/PatelDvip/`
- LinkedIn: `https://www.linkedin.com/in/dvip-patel-23320a230/`
- GitHub: `https://github.com/dvip1`

Keep this information consistent anywhere the author, social profiles, schema data, or metadata are edited.

## Analytics and Privacy

The GA4 measurement ID is configured in `hugo.toml`:

```toml
[services.googleAnalytics]
id = 'G-WLYGQMT86N'
```

Cookie consent is handled by `layouts/partials/cookie-consent.html`, which is included from `layouts/partials/extend_head.html`.

If analytics, cookies, embeds, or third-party scripts change, update `content/privacy-policy.md`.

## Deployment Notes

The privacy policy says the site is hosted on AWS Amplify. A typical deployment flow is:

1. Commit changes to content/config/layouts/static assets.
2. Let AWS Amplify run the Hugo build.
3. Verify the deployed site, sitemap, RSS feed, and key post URLs.

If deployment is configured to use committed `public/` output instead, run `hugo --gc --minify` before committing and include the generated changes intentionally.
