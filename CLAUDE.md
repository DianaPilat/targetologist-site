# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-page bilingual (Russian/English) marketing site for a Meta Ads specialist. It is plain static HTML/CSS/JS with no build step, no package manager, no tests and no linter. Everything served lives in `public/` and is deployed as static assets through Cloudflare Workers (`wrangler.jsonc`, `assets.directory = ./public`).

- Preview locally: `npx wrangler dev` (or open `public/index.html` directly).
- Deploy: `npx wrangler deploy`.

## Architecture

- `public/index.html` holds all markup plus one inline `<script>` at the bottom (language toggle, reveal-on-scroll, stat count-up). `public/privacy.html` is the only other page.
- `public/tokens.css` is the single source of truth for design tokens (OKLCH colors, type scale, spacing, motion). `public/styles.css` has the components and must read only from these variables, never hard-code colors, sizes or easings. Design rules are documented at the top of `tokens.css` (neutrals plus one willow-green accent, max 3 font families, 4px radius).
- Fonts (Literata, Onest) come from Google Fonts in `<head>`.

### Bilingual mechanism

Russian is the default text written directly in the markup. English lives in a `data-en` attribute on the same element (`data-en-aria` and `data-en-alt` for aria-label and alt). On load, the script copies the existing innerHTML into `data-ru`, and `setLang()` swaps `innerHTML` from `data-en` or `data-ru`. The choice is stored in `localStorage`. When adding or editing any visible text, update both the Russian content and its `data-en` value. Because `data-en` is swapped in as innerHTML, inline markup such as `<em>` is HTML-escaped inside the attribute. The page title and meta description are translated separately in the `meta` object in the script.

The stat count-up (`[data-to]`, `data-decimals`) rewrites the number inside the current language's string, so keep the number as the only digits-and-separators run it needs to replace.

### Other notes

- Motion: elements with `.reveal` fade in via IntersectionObserver, and the script sets a per-sibling `--i` stagger index. Timing comes from the `--t-*` and `--stagger` tokens, which `tokens.css` zeroes under `prefers-reduced-motion`. JS reads `--count-ms` from CSS.
- Analytics: the Meta Pixel (ID `3158561757676824`) is live in `<head>`. The HTML comment above it holds unused Plausible and pixel placeholder snippets (`yourdomain.com`, `YOUR_PIXEL_ID`) that are not active.
- The contact form posts via a `mailto:` action, so there is no backend.
- `portrait.jpg` is pre-cropped square, so CSS applies no focal-point offset.
