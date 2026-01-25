# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal tech notes site built with Hugo and the PaperMod theme. Posts are written in Markdown with TOML frontmatter.

## Commands

```bash
# Development server (includes drafts)
hugo server -D

# Production build
hugo --minify

# Create new post
hugo new posts/my-post-title.md
```

## Architecture

- **Theme**: PaperMod installed as git submodule in `themes/hugo-PaperMod`
- **Content**: All posts go in `content/posts/` as Markdown files
- **Custom layouts**: `layouts/` contains overrides for PaperMod
  - Mermaid diagram support via `_default/_markup/render-codeblock-mermaid.html` and `partials/extend_footer.html`
- **Deployment**: GitHub Actions workflow deploys to GitHub Pages on push to `main`

## Post Frontmatter

Posts use TOML frontmatter (`+++` delimiters). Key fields:
- `title`, `date`, `draft`, `tags`, `summary`
- `mermaid = true` to enable Mermaid diagrams in a post
- `ShowToc = false` to hide table of contents

## Adding Images to Posts

Use page bundles: create a folder with post name containing `index.md` and images:
```
content/posts/my-post/
├── index.md
└── diagram.png
```