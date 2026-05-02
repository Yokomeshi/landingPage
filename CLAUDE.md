# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Personal landing page for Jon Lexa, served from GitHub Pages at `jonlexa.com`. It is a single-page Jekyll site (one `index.html` rendered via the `base` layout) — there are no posts, no collections, and no JavaScript.

## Commands

Local development uses the `github-pages` gem so the build matches what GitHub Pages produces in production.

```bash
bundle install                    # install gems (first run / Gemfile.lock changes)
bundle exec jekyll serve          # serve at http://localhost:4000 with live rebuild
bundle exec jekyll build          # one-shot build into _site/
```

There are no tests, linters, or CI — pushes to the default branch are built and deployed by GitHub Pages.

## Architecture

The whole site is driven by the `compass:` namespace in `_config.yml`. Editing personal info (logo path, author name, tagline, social usernames, email) is done there, not in templates. Two boolean flags toggle optional sections:

- `compass.include_analytics` — when true, `_includes/head.html` pulls in `_includes/analytics.html` (empty placeholder; paste your GA snippet here).
- `compass.include_content` — when true, `index.html` pulls in `_includes/content.html` (empty placeholder; add free-form HTML/Markdown here).

Both include files are intentionally empty so the flags act as a kill switch without needing template edits.

Render path:

1. `index.html` declares `layout: base` and contains the card markup. Each section (logo, author, tagline, social links, content) is wrapped in a Liquid `{% if %}` against the matching `compass.*` value, so leaving a config field blank removes that section entirely.
2. `_layouts/base.html` is a thin shell that includes `_includes/head.html` and yields `{{ content }}`.
3. `_includes/head.html` references two stylesheets: `/assets/normalize.css` (vendored) and `/assets/main.css` (compiled by Jekyll from `assets/main.scss` — note the empty `---` front matter at the top of the SCSS file, which is what tells Jekyll to process it). The `main.css` link carries a `?v=N` cache-buster; bump it when shipping CSS changes.

## Conventions

- Domain config lives in three places that must agree: `CNAME` (GitHub Pages domain), and `url` / `enforce_ssl` in `_config.yml`.
- `exclude:` in `_config.yml` keeps repo metadata (`README.md`, `LICENSE`, `Gemfile*`, `CNAME`) out of `_site/`. Add new non-site files there.
- SCSS variables (`$bg`, `$text`, `$accent`, `$border`) at the top of `assets/main.scss` are the design tokens — restyle through these rather than hardcoding colors further down.

## Behavioral Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

Tradeoff: These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

These guidelines are working if: fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
