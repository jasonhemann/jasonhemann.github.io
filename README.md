# Hemann Personal Website

Personal site built with Jekyll and Minimal Mistakes, published at [hemann.pl](https://hemann.pl).

## Required Toolchain

For local development and testing, install:

1. Ruby `3.3.4` (see [`.ruby-version`](.ruby-version))
2. Bundler
3. Git

## Local Setup

Install Ruby dependencies:

```bash
bundle install
```

Run locally with live reload:

```bash
bundle exec jekyll serve --livereload
```

Then open: `http://127.0.0.1:4000/`

## Content Entry Points

The main hand-edited content is concentrated in a few files:

- [`about.md`](about.md): biography and background
- [`teaching.md`](teaching.md): teaching history and course-site links
- [`links.md`](links.md): curated reading/reference links
- [`_posts/`](_posts/): blog-style posts, if used
- [`_data/navigation.yml`](_data/navigation.yml): masthead navigation
- [`_config.yml`](_config.yml): site-wide metadata, author info, URL, and theme settings

## Typical Workflow

### 1) Edit page content

Update one of the top-level markdown pages or a post under [`_posts/`](_posts/).

### 2) Preview locally

Run `bundle exec jekyll serve --livereload` and review the affected pages.

### 3) Check course links and navigation

If you update teaching offerings or move a course site, review:

- [`teaching.md`](teaching.md)
- [`_data/navigation.yml`](_data/navigation.yml)
- any self-links that should remain root-relative under `https://hemann.pl`

## Notes

- This repo still carries some upstream Minimal Mistakes/theme-development files; treat [`about.md`](about.md), [`teaching.md`](teaching.md), [`links.md`](links.md), [`_data/navigation.yml`](_data/navigation.yml), and [`_config.yml`](_config.yml) as the main site-maintenance surface.
- The local Ruby/Jekyll toolchain is pinned in [`Gemfile`](Gemfile) and [`.ruby-version`](.ruby-version).
- The first `jekyll serve` on a new machine needs internet access to download the remote theme and/or bundled gems.
