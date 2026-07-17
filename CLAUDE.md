# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KAM Dynamics landing page — a static single-page site for a drone photography and videography
business (real estate, property inspections, construction progress) serving the greater Charlotte
area across NC and SC. Hosted on GitHub Pages at kamdynamics.com.

The page leads with aerial work. Tech-consulting work lives off-site at khary.net and is only
linked from the nav.

## Architecture

One page, no build system, no package manager, no dependencies. **All CSS and JS are inline in
`index.html`** — there are no external stylesheets or scripts beyond the Google Fonts link.

- **index.html** — the entire site. Roughly:
  - `<head>`: meta/OG tags, JSON-LD `LocalBusiness` structured data, the intro-card arming
    script, then one big `<style>` block
  - `<body>`: intro title card, ticker, nav, and sections in order — `#top` (hero),
    `#services-aerial`, `#preview` (the preview-before-flight workflow), `#editing`, `#about`,
    `#also`, `#contact`, footer
  - One `<script>` block at the bottom (~100 lines): nav scroll state, live timecode, portfolio
    flag, intro title card, footer end slate
- **images/** — brand and credential logos (SVG/PNG), `screenshots/` for the workflow montage,
  `og-card.jpg` (1200×630 social card)
- **videos/** — background clips and the logo sting
- **CNAME**, **robots.txt**, **sitemap.xml**, **favicon.svg**

Tracked files are only: `CLAUDE.md`, `CNAME`, `favicon.svg`, `index.html`, `robots.txt`,
`sitemap.xml`, plus assets. `direction-1-editorial.dropin.html` is an untracked design
exploration that the current page grew out of — it is not live, and not loaded by anything.

## Development

No build step. To preview locally:
```bash
python3 -m http.server 8000
# or
npx serve .
```

Changes deploy automatically via GitHub Pages when pushed to `main`.

## Design tokens

Defined in `:root` in `index.html`:

| Token | Value | Use |
|---|---|---|
| `--black` | `#050505` | page background |
| `--black-2` | `#0a0a0a` | raised surfaces |
| `--paper` | `#f2efe9` | primary text (warm off-white, not pure white) |
| `--paper-2` | `#d8d3c8` | secondary text |
| `--rule` | `rgba(255,255,255,0.12)` | borders on dark |
| `--rule-on-light` | `rgba(0,0,0,0.12)` | borders on light sections |
| `--red` | `#e63022` | brand red |
| `--red-bright` | `#ff3a2e` | hover/accent red |
| `--dim` | `#888880` | muted text |
| `--max` | `1400px` | content max width |

Note the red is `#e63022`, deliberately dialled back from pure `#FF0000` to read as editorial
rather than alarm-bell. Text is warm off-white, never `#fff`.

## Typography

- `--display` — **Archivo** (headings, body)
- `--display-narrow` — **Archivo Narrow** (large display type, footer copy)
- `--mono` — **IBM Plex Mono** (labels, tags, captions, nav links, timecode)

Loaded from Google Fonts with `preconnect` + `display=swap`.

## Feature flags

There is no config-flags block. The only runtime toggle is a URL param:

- `?portfolio=on` — sets `body[data-portfolio]` to reveal the portfolio section.

## Breakpoints

Only three media queries:

- `@media (prefers-reduced-motion: reduce)` — covers the intro card and scroll behaviour
- `@media (max-width: 1024px)` — layout collapses to single column; **nav links are hidden here
  and there is no hamburger replacement**, so only the brand and CTA remain on mobile
- `@media (max-width: 600px)` — the workflow steps go one-up

## The logo sting (intro card + footer end slate)

`videos/kam-logo-sting.mp4` is a ~4.1s clip where a light streak draws the KAM ellipse and the
logo resolves. Derived from `videos/kamdynamics-logo.mp4` (the 5s original, kept as the source)
by trimming ~0.9s of dead black off the front and cropping tight to the artwork.

Things worth knowing before touching it:

- **The clip is matted on pure `#000000`**, so `mix-blend-mode: screen` drops the background
  entirely. No alpha channel or transparent WebM is needed on any dark surface.
- **Its beats, in media seconds:** streak draws to ~1.6, the KAM mark resolves by ~2.1, a quiet
  beat to ~2.77, the wordmark fades in, and it **locks at 3.25** and holds for under a second.
  It plays at `playbackRate` 1.15 to buy ~0.4s.
- **The page fade starts at 2.55 — before the lock — and runs 1.3s**, so the logo finishes
  resolving while the page is already rising behind it (~1.2s of overlap). Starting early pays
  for the long fade: time-to-hero stays ~3.5s. `REVEAL_AT` and the CSS transition duration are
  coupled — `FADE` in the script must match `.intro`'s duration, and `FADE_FAST` must match
  `.intro--fast`. All are in media seconds against *this* cut, so if the asset is re-cut,
  re-measure the beats rather than guessing.
- **The backdrop and logo must fade together.** Fading the black out from under the logo sounds
  better but doesn't work: `screen` needs the black to composite against, and without it the
  clip's matte renders as an opaque black box.
- **The logo sits high, not centred** — `padding-bottom` on the grid puts its optical centre at
  ~42% of the viewport (geometric centre reads as low for a title card). It's padding rather than
  a translate on the video because `.intro--out` already owns the video's transform. A
  `max-height: 60vh` + `contain` on the video keeps a tall logo off the edges on short/landscape
  viewports, where width alone let it fill ~84% of the height.
- **The intro card is `display: none` unless JS arms it.** A synchronous script in `<head>` adds
  `intro-armed` + `intro-lock` to `<html>` only when the visitor has not seen it this session
  (`sessionStorage['kam-intro-seen']`) and does not prefer reduced motion. No-JS visitors get the
  hero, never a black overlay.
- **Any input skips it** (click/touch/key/wheel/scroll) using a fast 260ms fade — don't make
  someone sit through the unhurried version of a thing they just dismissed. Every failure path —
  autoplay blocked, video error, stall — must fall through to the hero, and all of them take the
  fast fade too. `BAIL_OUT` (3100ms from script start) is the backstop that catches a video which
  never plays; it is deliberately kept clear of the ~2.6s normal cue so a slow-starting video
  doesn't get its logo chopped by a watchdog racing the happy path.
- Handlers are wrapped rather than bound straight to `reveal` (`onSkip`/`onEnd`/`onErr`): bound
  directly, each would pass its Event to `reveal(fast)`, and an object is truthy — so the gentle
  fade would silently never run.
- The card has **no poster on purpose**: `images/kam-logo-still.png` is the *end* of the
  animation, so it would spoil the reveal and become an LCP candidate. The still is the footer
  poster and the reduced-motion fallback only.
- The footer end slate plays once per page view via `IntersectionObserver`, which `unobserve`s
  before calling `play()`.

## Conventions and gotchas

- **Screenshots are information, not decoration.** The `#preview` montage crops with
  `object-fit: cover` on desktop for a tidy 3-up grid, but the single-column mobile layout uses
  `height: auto` so nothing is cut off — `03-customer-package.png` is portrait (0.78 aspect) and
  a fixed height hides most of the document. A `max-height` cap plus `contain` keeps it bounded
  on wide tablet columns.
- The contact form posts to **FormSubmit.co** (`action` on the `.form` element).
- The nav mark stays the static `images/kam-logo.svg`. The sting is deliberately *not* used
  there: `.brand-mark` is 48×28 and sits beside the text "KAM Dynamics", so the clip's own
  wordmark would be illegible and redundant at that size.
- `images/kam-logo-animated.gif` is a 2.4 MB GIF of the same sting — superseded by the 111 KB
  MP4 and unreferenced. Don't use or commit it.

## Known gaps

- Nav links vanish at ≤1024px with no mobile menu to replace them.
- `prefers-reduced-motion` covers the intro card and scroll behaviour, but several
  unconditional animations still have no opt-out: `sun-pulse`, the `.scroll-cue` bob, the
  ticker marquee, the two looping background videos, and a 42ms `setInterval` driving the hero
  timecode.
