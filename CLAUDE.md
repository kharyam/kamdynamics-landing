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
  - `<head>`: meta/OG tags, JSON-LD `LocalBusiness` structured data, the intro-cut arming
    script, then one big `<style>` block
  - `<body>`: nav and sections in order — `#top` (hero, with the intro-cut logo slate inside it),
    `#services-aerial`, `#preview` (the preview-before-flight workflow), `#editing`, `#about`,
    `#also`, `#contact`, footer. (A hidden `.ticker` element still exists in the markup but is
    `display:none` — see below.)
  - One `<script>` block at the bottom (~90 lines): nav scroll state, live timecode, portfolio
    flag, the in-hero intro cut, footer end slate
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

- `@media (prefers-reduced-motion: reduce)` — disables smooth scroll (and, by never arming
  `.hero-cut`, the intro cut)
- `@media (max-width: 1024px)` — layout collapses to single column; **nav links are hidden here
  and there is no hamburger replacement**, so only the brand and CTA remain on mobile
- `@media (max-width: 600px)` — the workflow steps go one-up

## The logo sting (in-hero intro cut + footer end slate)

`videos/kam-logo-sting.mp4` is a ~4.1s clip where a light streak draws the KAM ellipse and the
logo resolves. Derived from `videos/kamdynamics-logo.mp4` (the 5s original, kept as the source)
by trimming ~0.9s of dead black off the front and cropping tight to the artwork.

**The intro is an in-hero cut, not a full-screen overlay.** The hero (`#top`) renders fully
composed from the first frame — nav links, CTA, the REC/timecode HUD, the meta row, the lede, and
both buttons are all live and clickable. Only two things wait: the hero **background** and the
**headline**. The logo plays as the background footage the page sits on (`.hero-slate`, a video
letterboxed in the upper band on opaque black), then at the sting's lock it cross-dissolves to the
drone footage (`.hero-video`) while the three `#hero-h1 .hl` headline lines stage in one at a time.
The nav brand (`.brand`) also holds until the cut, so the wordmark isn't shown twice while the
logo plays. An earlier full-screen `.intro` overlay approach was removed — don't reintroduce it.

Things worth knowing before touching it:

- **The clip is matted on pure `#000000`**, so `mix-blend-mode: screen` drops the background
  entirely. The slate is dimmed to `opacity: 0.6` (about the house video's 0.55) so the headline
  reads over it and there's no brightness pop at the cut.
- **Its beats, in media seconds:** streak draws to ~1.6, the KAM mark resolves by ~2.1, a quiet
  beat to ~2.77, the wordmark fades in, and it **locks at 3.25** and holds for under a second.
  It plays at `playbackRate` 1.15. The JS cues the cut at `CUT_AT = 3.5` (just past the lock);
  this is in media seconds against *this* cut, so re-measure the beats if the asset is re-cut.
- **`.hero-cut` on `<html>` is what arms everything.** A synchronous `<head>` script adds it only
  on the first visit of a session (`sessionStorage['kam-intro-seen']`) with motion allowed. All
  the waiting-and-staging CSS is scoped under `.hero-cut`, and the class is set *before first
  paint* so there's no flash of the un-staged headline. Without it — repeat visit, reduced motion,
  or no-JS — the hero renders fully composed immediately: `.hero-slate` stays `display:none`, the
  headline lines and brand are visible, the house video plays. `is-live` (added by the JS at the
  cut) triggers the dissolve + stagger.
- **Every failure path falls straight to the live hero.** Autoplay blocked (`play().catch`),
  decode/source error, and a stall that never reaches `CUT_AT` all call `goLive()`; a `BAIL`
  watchdog (4200ms) is the final backstop. Any user interaction (`click/touch/key/wheel/scroll`)
  also resolves the cut at once — so a visitor who immediately scrolls never leaves the headline
  un-staged. Nothing can strand the logo on screen or leave the headline hidden.
- **`CUT_AT`, the `.hero-slate` transition (0.6s), and the `#hero-h1 .hl` stagger delays are
  coupled.** The slate's dissolve and the headline stagger both fire off `is-live`; keep the
  `setTimeout` that removes the slate (700ms) past the dissolve.
- **The slate sits in the upper band** (`top: 8%; height: 46%; object-fit: contain`), not the full
  frame: the headline is huge and bottom-aligned, so a full-height logo dangled its wordmark into
  "ANGLE." and stacked black above. Anchoring it high clears the lower title and removes the dead
  space overhead.
- **`images/kam-logo-still.png`** (the resolved logo) is used as the **footer** slate poster and
  reduced-motion fallback only — not by the hero cut (the hero starts on the moving logo, not its
  end frame).
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
  wordmark would be illegible and redundant at that size. (On first visit the whole `.brand`
  fades in after the logo cut — see the intro-cut section.)
- **The trust ticker is hidden** (`display:none` on `.ticker`; the markup is kept for easy
  revival). Its scrolling marquee read as dated. When it went, its "Fully insured" signal was
  re-added as the first bullet in the `#services-aerial` feature list; "Now booking" was dropped
  and "Fast turnaround" already existed in that list. If you re-enable it, restore `.nav`'s
  `top` offset (it was moved to `0` to fill the 32px the fixed ticker used to occupy).
- `images/kam-logo-animated.gif` is a 2.4 MB GIF of the same sting — superseded by the 111 KB
  MP4 and unreferenced. Don't use or commit it.

## Known gaps

- Nav links vanish at ≤1024px with no mobile menu to replace them.
- `prefers-reduced-motion` gates the intro cut and smooth scroll, but several unconditional
  animations still have no opt-out: `sun-pulse`, the `.scroll-cue` bob, the two looping background
  videos, and a 42ms `setInterval` driving the hero timecode.
