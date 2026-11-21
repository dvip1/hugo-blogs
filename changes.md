# Planned Changes

Backlog of ideas for the blog's design, content, and tooling. Pick items from here, and move them to **Done** with the date and commit when shipped.

Design reference: the [fly.io blog](https://fly.io/blog/sqlite-internals-btree/) (type, spacing, margin notes, illustrations).

## Design & Typography

- [ ] **Code block header**: show the filename or language label above code blocks (copy button already exists).
- [ ] **Callout boxes**: shortcode for "Note", "Gotcha", "Why I did this" callouts, styled like fly.io's speech-bubble asides.
- [ ] **Self-host fonts**: serve Fraunces, Bricolage Grotesque, and Fragment Mono from `static/fonts/` instead of Google Fonts (faster first paint, no third-party request; update the privacy policy if done).
- [ ] **Light-mode texture**: consider a subtle top glow or paper tint in light mode to match the dark-mode glow.
- [ ] **Link hover for post list**: review hover states on the home list and tag pages for consistency with post links.

## Images & Illustrations

- [ ] **Redraw the Real-Time Monitoring architecture diagram** in the site palette (green/cream) so it fits both themes.
- [ ] **Hero image per post**: one consistent style (duotone diagram or simple line illustration) for every post; also improves social previews.
- [ ] **Open Graph images**: add `cover`/`images` front matter per post so shares on X/LinkedIn show a proper card.
- [ ] **Image formats**: convert post images to `.webp` under `static/images/posts/<slug>/`.

## Content & Structure

- [ ] **Descriptive section headings** (needs author approval per heading), e.g. "First Phase" → "Ingesting RTSP Camera Streams with MediaMTX", "Problèmes" → "Problems I Hit (and Fixes)". Keep playful lines as the first sentence.
- [ ] **Add margin notes** to existing posts where a side comment helps (Real-Time Monitoring: MediaMTX formats, Problèmes).
- [ ] **Related posts**: link between the Text-to-SQL and Real-Time Monitoring write-ups where relevant.
- [ ] **Short direct-answer intro** at the top of each post (what was built, for whom, stack), per CLAUDE.md conventions.

## SEO & Discoverability

- [ ] **Structured data check**: verify PaperMod's JSON-LD (BlogPosting, author, dates) renders correctly with Google's Rich Results Test.
- [ ] **Per-post keywords/descriptions audit**: make sure every post has `description`, `summary`, `keywords`, tags, categories.
- [ ] **About page**: a short author page (Dvip Patel, GitHub `dvip1`, X `PatelDvip`, LinkedIn) for author identity.

## Tooling & Maintenance

- [ ] **Decide on committing `public/`**: either commit builds intentionally or build in CI/hosting and stop tracking `public/`.
- [ ] **Clean stale generated CSS**: `hugo --gc` leaves old `public/assets/css/stylesheet.*.css` files behind; remove unreferenced ones or rebuild into a clean `public/`.
- [ ] **Fix PaperMod deprecation warnings** (`.Language.LanguageDirection`, `.Language.LanguageCode`) by updating the theme submodule.

## Done

- 2026-11-21: Removed the island contour graphic from the home intro (`076a9d8`).
- 2026-11-21: Fraunces + Bricolage Grotesque + Fragment Mono type system, fly.io-style rhythm and links (`56a2101`).
- 2026-11-21: Post sections promoted to H2, collapsible ToC, dark-mode palette (ink green, cream headings), themed cookie banner (`60912a7`).
- 2026-11-21: Light mode as default theme; `aside` margin-note shortcode.
