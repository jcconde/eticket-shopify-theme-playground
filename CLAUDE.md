# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Shopify theme for the "eticket" store (`bfcbeef`), built on top of **Dawn 15.4.0** (see `config/settings_schema.json`) with a legacy Bootstrap 3 / jQuery 3.2.0 "eticket" design layered on top. There is no build step, package manager, linter or test suite: files under `assets/`, `sections/`, `snippets/`, etc. are what gets uploaded to Shopify as-is.

## Commands

Shopify CLI is the only tooling. Environments are defined in `shopify.theme.toml` (git-ignored, but present locally):

- `default` → **staging** theme (`134755352639`). This is what you get without `--environment`.
- `production` → live theme (`134735626303`). Release only; never push here unless explicitly asked.

```bash
shopify theme dev                         # local preview against the staging theme
shopify theme push                        # push to staging (default env)
shopify theme pull                        # pull staging (see "Theme editor" caveat below)
shopify theme check                       # Liquid/theme lint
shopify theme push --environment production   # release only
```

## Architecture

**Layering of Dawn + eticket.** `layout/theme.liquid` loads Dawn's own assets (`base.css`, `component-*.css`, `global.js`, ...) and then, in order, the legacy stack: `vendor-bootstrap*.css`, `vendor-core.css`, `eticket-theme.css`, `eticket-theme-responsive.css`, followed (before `</body>`) by jQuery 3.2.0 and the `vendor-*` jQuery plugins (slider, select, flexslider, countdown, imagemapster, tooltip, bootstrap). Notes:

- Asset naming convention: `vendor-*` = third-party libraries vendored in; `eticket-*` = site-specific theme CSS/fonts/images; everything else is stock Dawn. Custom styling goes in `eticket-theme.css` (and `eticket-theme-responsive.css` for breakpoints), not in Dawn's `base.css`/`component-*.css`.
- Because Dawn's CSS and Bootstrap 3 coexist, selector collisions are the main source of visual bugs. Check cascade order in `theme.liquid` before assuming a rule is wrong.
- Vendored JS references images by absolute CDN URL (e.g. `vendor-tooltip.js`) rather than via `asset_url`; keep that convention.

**Custom sections vs. Dawn sections.** Eticket-specific sections are plain Liquid with Bootstrap markup (`container`/`row`), e.g. `hero-banner`, `upcoming-events`, `today-schedule`, `event-by-categories`, `latest-news-and-tweets`, `sponsors`, `stats`, `cta-learn-more`, `top-header`, `video-parallax`, `recent-videos`. Each carries its own `{% schema %}` (settings/blocks/presets). Dawn sections (`main-*`, `header`, `footer`, `featured-*`, `image-banner`, ...) remain mostly stock. Some custom sections currently contain static placeholder markup (hard-coded dates/content) alongside their schema settings — check before assuming a field is wired up.

**Templates and groups.** `templates/*.json` (and `templates/customers/*.json`) compose sections per page type; `sections/header-group.json` and `footer-group.json` compose the header (`top-header` + `header` + announcement bar) and footer. `top-header` is an eticket addition inside the header group.

**Theme editor caveat.** `templates/*.json`, `sections/*-group.json` and `config/settings_data.json` are rewritten by the Shopify theme editor (the files say so in a header comment). Expect them to drift from git if someone edits in the admin; run `shopify theme pull` (staging) before hand-editing them to avoid overwriting merchant changes. Keep each section entry's `settings` object present (even `{}`) — recent commits do this deliberately for schema consistency.

**Localization.** `locales/*.json` (storefront strings) and `locales/*.schema.json` (editor labels) come from Dawn; `en.default*.json` is the source. Schema labels use `t:` keys, but custom eticket sections tend to use literal English strings.

## Conventions

- Commit messages are short imperative sentences naming the file/section touched, with identifiers in backticks (e.g. ``Add styling for "Product Detail Page" in `eticket-theme.css` ``).
- `.shopify/`, `shopify.theme.toml`, `.idea/` and `.env*` are git-ignored; theme IDs and store handle live only in the local toml.
