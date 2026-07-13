# chengfu0118.github.io

Cheng Fu's personal academic website — a Jekyll site built on the
[academicpages](https://github.com/academicpages/academicpages.github.io) theme
(a detached fork of [minimal-mistakes](https://github.com/mmistakes/minimal-mistakes)).

Live at <https://chengfu0118.github.io>. Pushing to `master` triggers the
**Build and deploy Jekyll site to Pages** GitHub Action
([`.github/workflows/pages.yml`](.github/workflows/pages.yml)), which builds with
the pinned `Gemfile.lock` and deploys to GitHub Pages.

## Content

- `_pages/about.md` — landing page (bio, research interests, work experience)
- `_pages/publications.md` + `_publications/*.md` — one file per paper
- `_pages/cv.md` — CV page
- `_data/navigation.yml` — top nav
- `_config.yml` — site-wide settings
- `images/hq.JPG` — avatar; `files/` — linked assets (e.g. slides)

## Styling

Custom appearance lives in [`assets/css/custom.css`](assets/css/custom.css), loaded
after the theme's compiled CSS via [`_includes/head/custom.html`](_includes/head/custom.html)
(includes automatic dark mode). Theme internals under `_sass/`, `_layouts/`, and
`_includes/` are upstream — prefer overriding in `custom.css`.

## Local preview

There is no local Ruby toolchain; build inside Docker. See [`CLAUDE.md`](CLAUDE.md)
for the exact Docker build/preview recipe.
