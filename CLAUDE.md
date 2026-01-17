# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KAM Dynamics landing page - a static website for a technology consulting and aerial videography business. The site is hosted on GitHub Pages at kamdynamics.com.

## Architecture

This is a single-page static website with no build system or package manager:

- **index.html** - Main landing page with all CSS/JS inline. Contains:
  - Navigation, hero section, services, portfolio placeholder, contact form
  - Configuration flags at the top of the `<script>` section for toggling features
  - Animations: page loader, typewriter effect, particles, parallax scrolling, scroll reveals
  - Contact form uses FormSubmit.co for backend processing

- **aboutme.html** - Loaded via iframe in a modal. Contains:
  - Bio, education, and certifications with card-based layout
  - Configuration flags in its own `<script>` section

- **images/** - SVG and PNG assets for logos (Red Hat, AWS, CNCF, FAA, etc.)

- **CNAME** - GitHub Pages custom domain configuration

## Key Configuration Flags

In `index.html` script section (around line 1520):
- `SHOW_PORTFOLIO` - Toggle portfolio section visibility
- `SCROLL_ANIMATIONS_REVERSE` - Enable/disable reverse animations on scroll up
- `ENABLE_LOADING_ANIMATION` / `LOADING_DURATION` - Page loader settings
- `ENABLE_PARTICLES` / `PARTICLE_COUNT` - Hero particle animation
- `ENABLE_TYPEWRITER` / `TYPEWRITER_SPEED` - Hero title typewriter effect
- `ENABLE_HERO_PARALLAX` - Hero text parallax on scroll

In `aboutme.html` script section:
- `ENABLE_STATS_COUNTER` - Toggle certification stats display
- `PARALLAX_INTENSITY` - Certification card parallax (0 to disable)
- `PROFESSIONAL_START_YEAR` - For calculating years of experience

## Development

No build step required. To preview locally:
```bash
python3 -m http.server 8000
# or
npx serve .
```

Changes deploy automatically via GitHub Pages when pushed to main branch.

## CSS Custom Properties

Design tokens defined in `:root`:
- `--kam-red: #FF0000` - Primary brand color
- `--kam-black: #0a0a0a` - Background color
- `--kam-white: #ffffff` - Text color
- `--kam-gray: #1a1a1a` - Secondary background
- `--kam-light-gray: #333333` - Borders/accents
- `--transition-smooth` - Standard cubic-bezier transition

## Typography

- Headings: Orbitron (Google Fonts)
- Body: Work Sans (Google Fonts)
