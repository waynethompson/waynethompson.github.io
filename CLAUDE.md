# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Jekyll 4.3.3 static site (Wayne Thompson's blog, "Code, Cloud, & CTO Insights"), deployed via GitHub Pages (see `CNAME`). Styling is Bootstrap 5 (loaded via CDN in `_layouts/default.html`) on top of the `minima` gem theme.

## Commands

```bash
bundle install                          # install gems (first-time setup)
bundle exec jekyll serve -w --incremental   # run local dev server with live rebuild
bundle update jekyll                    # upgrade jekyll
```

There is no test suite, linter, or build step beyond Jekyll's own site generation — `bundle exec jekyll build` (implicit in `serve`) is the only "build". `_site/` is the generated output and is gitignored; never hand-edit it.

## Architecture

**Layout chain**: every page sets `layout:` in its front matter, and layouts themselves chain to `default`:
- `default.html` — outer HTML shell: `head.html`, `nav.html`, main content column + `sidebar.html`, `footer.html`. All other layouts render into `{{ content }}` here.
- `post.html` — blog post article. Renders title/date/author meta, post content, tags (via `post-tags.html`), Disqus comments (only when `jekyll.environment == "production"` and `page.comments != false`), and a "related posts" list.
- `home.html` — the index page's post listing, with pagination support (`site.paginate`) and excerpt display (`site.show_excerpts`).
- `page.html` — plain content page (used by `about.markdown`, `categories.html`, `tags.html`).
- `tag_index.html` — auto-generated tag archive page (see plugin below); lists all posts for one tag.

**Custom tag pages plugin** (`_plugins/jekyll-tag-pages.rb`): a Jekyll `Generator` that creates one page per tag at build time, using the `tag_index` layout. Output directory is controlled by `_config.yml`'s `tag_dir` setting, which is currently `tage_index` (not a typo to "fix" — `_includes/sidebar.html` and `_includes/post-tags.html` both hardcode links to `/tage_index/<slug>/`, so changing `tag_dir` requires updating those includes too).

**Categories vs. tags**: both are standard Jekyll front-matter taxonomies (`site.categories`, `site.tags`), but they're surfaced differently:
- Categories only get a single aggregate page: `categories.html` (permalink `/categories/`), grouping all posts by category inline.
- Tags get both an aggregate page (`tags.html`, permalink `/tags/`) *and* individual per-tag archive pages via the plugin above.
- `_includes/sidebar.html` lists all categories and the first 18 tags (sorted) as quick links.

**Post front matter conventions** (see any file under `_posts/` for a live example):
```yaml
layout: post
title:  "..."
date:   2026-08-05 12:00:00 +1000
categories: [javascript, tooling]
tags: [javascript, typescript, npm, ...]
permalink: /blog/2026/Choosing-Your-JavaScript-Package-Manager/
```
Permalinks are set explicitly per post rather than relying on Jekyll's default `/year/month/day/title` pattern — new posts should follow the `/blog/<year>/<Post-Title>/` convention seen in recent posts. `<!--more-->` marks the excerpt cutoff (configured as `excerpt_separator` in `_config.yml`).

**`_posts/` directory structure is organizational only** (e.g. `_posts/2026/Q3/...`) — Jekyll determines post date/ordering from the filename's leading `YYYY-MM-DD`, not from the folder path, so subfolders can be nested however is convenient without affecting the build.
