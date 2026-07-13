# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Cheng Fu's personal academic website — a Jekyll site built on the **academicpages** theme
(a detached fork of **minimal-mistakes**). It is served by **classic GitHub Pages**, which
builds the site server-side from the **`master`** branch (there is no `main` branch and no
build/deploy GitHub Action). Pushing to `master` deploys to https://chengfu0118.github.io.

The repo's history is **unrelated to upstream academicpages** (no common ancestor), so you
cannot `git merge`/"Sync fork" from upstream — porting upstream changes means copying files,
not merging. See "Relationship to upstream" below.

## Build & preview (no Ruby on the machine — use Docker)

There is no local Ruby/Jekyll toolchain; build inside Docker with the `github-pages` gem
(what GitHub Pages actually uses). `vendor/`, `_site/`, `.bundle/`, `Gemfile.lock`, and
`_config.preview.yml` are gitignored.

Build:
```bash
docker run --rm -v "$PWD":/srv -w /srv ruby:3.3 bash -lc '
  gem install bundler -N
  bundle config set --local path vendor/bundle
  bundle install
  bundle exec jekyll build --config _config.yml,_config.preview.yml
'
```
- Build with the extra `_config.preview.yml` (gitignored, `url: ""`) so asset links are
  **root-relative** and the output works under any static server. Do NOT use `jekyll serve`
  for the shared preview: it rewrites `site.url` to the bind host, so assets point at
  `http://0.0.0.0:4000` and break.
- `_config.yml` is NOT reloaded between builds if you keep a process alive — rebuild fully.

Preview (serve `_site` on :4000 with clean URLs, matching GitHub Pages' extensionless
`.html` serving):
```bash
docker run -d --name cf-preview -p 4000:4000 -v "$PWD/_site":/site:ro python:3-alpine \
  python -c "import http.server,os;os.chdir('/site');\
h=http.server.SimpleHTTPRequestHandler;\
tp=h.translate_path;\
h.translate_path=lambda s,p:(lambda r: r if os.path.exists(r) or not os.path.exists(r+'.html') else r+'.html')(tp(s,p));\
http.server.test(HandlerClass=h,port=4000,bind='0.0.0.0')"
```
After rebuilding `_site`, recreate the container (`docker rm -f cf-preview` then re-run) —
the bind mount goes stale if `_site` was deleted and regenerated.

## Architecture

### Content vs. theme (what to touch)
- **Content** (the owner's material — edit deliberately): `_pages/about.md` (bio, Research
  Interests, Work Experience), `_pages/cv.md`, `_pages/publications.md`, `_publications/*.md`
  (one file per paper), `_data/navigation.yml` (top nav), `images/` (avatar `hq.JPG`),
  `files/`, and personal values in `_config.yml`.
- **Theme internals** (from upstream; avoid editing directly): `_layouts/`, `_sass/`,
  `assets/css/main.scss`, `assets/js/`, most of `_includes/`.

### The visual refresh is an overlay — this is the key pattern
All custom styling lives in **`assets/css/custom.css`**, loaded *after* the theme's compiled
`main.css` via the `<link>` in **`_includes/head/custom.html`**. Do NOT edit theme `_sass` for
appearance changes — add/adjust rules in `custom.css` so the work survives theme updates.

`custom.css` provides: modern typography/accent (classic blue), section-header styling,
circular avatar + sidebar, publication cards, the Work-Experience timeline, and
**automatic dark mode** via `@media (prefers-color-scheme: dark)`. Dark mode works by
re-pointing BOTH this overlay's own tokens AND the **theme's `--global-*` skin variables**
(e.g. `--global-bg-color`, `--global-masthead-link-color`) — the theme's built-in dark skin
is gated behind a manual toggle which we hide (`#theme-toggle { display: none }`), so overlay
dark mode is the single source of truth.

### Notable template customizations (in `_includes/`)
- `archive-single.html` — publication titles link **directly to the paper** (`post.paperurl`,
  new tab) instead of an intermediate page; venue rendered as a chip (`.archive__item-venue`).
  Each `_publications/*.md` therefore needs a real `paperurl`.
- `head/custom.html` — loads FontAwesome + Academicons and the `custom.css` overlay.
- `seo.html` — adds an `og:image` fallback from `site.og_image` (the stock theme only feeds
  `twitter:image`).
- `masthead.html` — brand link uses `site.masthead_title | default: site.title` (set to
  "Home" so the nav isn't a redundant name; `site.title` stays "Cheng Fu" for the tab/SEO).
- `footer.html` — trimmed to just copyright + "last updated" (Follow/Feed/Sitemap removed).

### `_config.yml` specifics
`site_theme: "default"`, `analytics.provider: false`, `og_image: "hq.JPG"`,
`atom_feed.hide: true`, `masthead_title: "Home"`, and `vendor`/`.bundle` in `exclude` (else
Jekyll scans the vendored gems and the build fails). Collections: publications + portfolio.

## Gotchas
- **`bundle install` fails naively.** The `github-pages` gem is version-sensitive: a stale
  `Gemfile.lock` pins an ancient bundler that calls `String#untaint` (removed in Ruby 3.2),
  and Ruby 2.7 can't resolve a compatible nokogiri. The working path is `ruby:3.3` with the
  lock deleted (as above). Never commit `Gemfile.lock` (gitignored).
- **No `package.json`.** It was removed to clear Dependabot alerts; `assets/js/main.min.js` is
  committed and served as-is (no npm build step). Don't reintroduce npm tooling.
- **Publication dates are unreliable.** Most `_publications/*.md` have missing/placeholder
  `date:` (`2010-10-01`); the real year lives in the `venue:` string. Do not sort/group by date.
- **Setext headings.** In `about.md`, `text` underlined with `======` renders as an `<h1>`
  (not `<h2>`); the section-header CSS targets `.page__content h1, .page__content h2`.
- `_config.preview.yml` is a local, gitignored preview override (`url: ""`) — never rely on
  it for production; production uses `_config.yml`.

## Relationship to upstream
Upstream is `academicpages/academicpages.github.io`. Because histories are unrelated, adopt
upstream improvements by **copying files** (theme dirs, `markdown_generator/`, tooling) onto
this tree and re-applying the customizations above — not via `git merge`.

## Conventions
- **Do not add a `Co-Authored-By: Claude ...` trailer to commits** (owner's org disallows it).
- Commit/push only when asked; `master` is the live branch, so a push deploys immediately.
- Treat bio/publications/CV as the owner's material — confirm before rewriting content;
  formatting/theme/tooling changes are fair game.
