# crewai-docs — the CrewAI Field Guide

A concept-level field guide to the [CrewAI](https://github.com/crewAIInc/crewAI) codebase,
focused on **execution semantics** — what actually happens when a crew or flow runs.
**Written and maintained by [Doc Holiday](https://doc.holiday)**. Each page explains why a
subsystem exists, how it behaves at runtime, and how it relates to its neighbors —
complementing the official docs at [docs.crewai.com](https://docs.crewai.com/).

Built with [Hugo](https://gohugo.io/) and the [E25DX](https://github.com/dumindu/E25DX)
theme (site skeleton ported from the dspy-docs field guide), deployed to **GitHub Pages**
via `.github/workflows/gh-pages.yml`.

## Prerequisites

- [Hugo **extended**](https://gohugo.io/installation/) — CI pins `HUGO_VERSION` in
  [`.github/workflows/gh-pages.yml`](.github/workflows/gh-pages.yml). It is held at
  **0.155.0** because newer Hugo emits deprecation warnings from the E25DX theme (harmless
  locally); bump it alongside a theme update.
- `git` (the theme is a submodule).

## Local development

```bash
git clone --recurse-submodules https://github.com/sandgardenhq/crewai-docs.git
cd crewai-docs

# If you already cloned without --recurse-submodules:
git submodule update --init --recursive

# Dev server with live reload at http://localhost:1313
hugo server

# Production build (output in ./public)
hugo --gc --minify
```

## Content organization

Doc Holiday writes the guide pages as plain markdown; they live under `content/en/docs/`
with numbered filenames (`00-…` to `09-…`) that fix the reading order and keep the pages'
relative cross-links (`./NN-slug.md`) resolvable — the render-link hook rewrites them to
page URLs at build time.

Each page carries front matter (`title`, `url`, `description`). The sidebar is defined in
`data/en/docs/sidebar.yml`; the theme builds each sidebar link as `/docs/<urlize(title)>/`,
so **sidebar titles must urlize to the same slug as the page's `url` front matter**.

Home-page blocks are data-driven: `data/en/home/{hero,bento,card-grid}.yaml`.

## Deployment

Pushes to `main` build with pinned Hugo and deploy to GitHub Pages (Settings → Pages →
Source: GitHub Actions). The `gh-pages.yml` workflow injects the real Pages base URL
(including the `/crewai-docs/` project subpath) at build so all relative links resolve.
