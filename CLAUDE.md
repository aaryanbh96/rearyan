# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`rearyan` is Aryan Bhardwaj's personal site (rearyan.com) — hand-written static HTML with **no build step, no package manager, no dependencies, and no tests**. There is no `package.json`, no bundler, and no CI config. You edit `.html` files and they are what ships.

## Commands

```bash
# Local preview (any static server works — the site is plain files)
python -m http.server 8000     # then open http://localhost:8000/index.html

# Deploy = push to main; there is no build
git push origin main
```

There is no lint, test, or build command. "Verifying a change" means opening the page in a browser and looking at it — including at mobile widths, since every page carries four breakpoints.

Deployment is configured outside the repo (no CNAME, workflow, or deploy config is tracked here).

## Architecture: every page is self-contained

This is the single most important fact about this codebase. Each top-level `.html` file inlines **its own complete `<style>` block and its own `<script>` block**. There is no shared stylesheet and no shared script. The only runtime dependency any page loads is Google Fonts.

Consequences that will bite you:

- **`header.html` and `footer.html` are orphaned.** They exist at repo root and look like partials, but *nothing fetches them*. Editing them changes nothing on the live site. `bookshelf.html` and `future.html` even contain a bare `<div id="footer-placeholder"></div>` with no loader — a vestige of an abandoned include pattern.
- **`assets/css/main.css`, `assets/css/animations.css`, `assets/js/main.js`, `assets/js/intro.js`, and `assets/js/three.min.js` are all 0 bytes and referenced by nothing.** Do not add code to them expecting it to load.
- **`contact.html` is 0 bytes and unlinked.** The real contact form lives at `index.html#contact` (Formspree endpoint, submitted via `fetch` in the inline script).

**Therefore: a change to the nav, footer, or shared visual language must be repeated by hand in every page that shows it** (`index.html`, `projects.html`, `resume.html`, `bookshelf.html`, `future.html`). When asked to change something "site-wide," enumerate the pages and edit each one — and say which pages you touched.

## The pages

| File | Role |
|---|---|
| `index.html` | Home — intro animation, hero video, Selected Work cards, About, contact form |
| `projects.html` | Despite the name, one flagship case study (Tao of Tea Operations Suite) plus data/research sections |
| `resume.html` | Web resume, mirrors `assets/Aryan_Bhardwaj_Resume.pdf` — **keep the two in sync when editing either** |
| `bookshelf.html` | Annotated papers + books list; the only page with a library video background |
| `future.html` | "Thoughts" — a Coming Soon stub |
| `faire-analysis.html` | 1.79 MB **generated** Jupyter `nbconvert` export (title `executed_final`) |
| `tao-catalog.html` | Untracked and orphaned; deliberate light "paper" theme outlier |

`faire-analysis.html` is machine-generated — do not hand-edit it. Changes belong in the source notebook (not in this repo), re-exported.

## Design system (duplicated, not shared)

Black `#000` background, white text, `Space Grotesk` for body, `JetBrains Mono` for the `re:aryan` logo and mono accents, `#64c8ff` as the one link accent. `.container` is `max-width: 1200px` — except `resume.html` (1000px) and the `projects.html` case study (880px). Breakpoints are 1024 / 768 / 480 / 360.

Nav is five links in fixed order (Selected Work, Resume, Thoughts, Bookshelf, Get in Touch). The current page's link gets `class="nav-btn highlight"`. "Get in Touch" points to `index.html#contact` on subpages and `#contact` on index. The logo and nav fade to `opacity: 0.1` past 100px of scroll via an inline listener.

**Mobile styles live in a second `<style>` block appended after the main one**, labeled `/* ═══ UNIFIED MOBILE PATCH (applies last, wins cascade) ═══ */`. It is `!important`-heavy and re-docks the nav into a blurred bottom pill under 768px. Two variants of it exist (a long one in `projects.html`/`resume.html`, a condensed one in `bookshelf.html`/`future.html`). **Mobile fixes go in that block** — editing the main block will lose the cascade fight.

## The intro animation (`index.html` only)

A phase machine (`showColon` → `typeAryan` → `merge` → `flicker` → `fadeToSite`) types out `re:aryan` over a full-screen loading overlay. It is gated on `sessionStorage.getItem('introPlayed')`, with a pre-paint inline script in `<head>` that sets a `.skip-intro` class to avoid a flash on repeat visits. It also skips when `location.hash` is present, auto-skips after a 3s failsafe, and skips on any click/touch/keypress.

**To see the intro while testing, clear sessionStorage** (or use a fresh private window) and load `index.html` with no hash.

## Known rough edges

Treat these as pre-existing, and don't "fix" them incidentally while doing unrelated work — but do flag them if a task lands nearby:

- All pages sit flat at repo root, so **same-site links should be bare relative paths** (`faire-analysis.html`, `projects.html`). Absolute `https://rearyan.com/...` links jump to production from a local preview — don't reintroduce them. `mailto:ab@rearyan.com` is unaffected.
- `bookshelf.html` and `future.html` have **no `</head>` and no `<body>` tag** — all markup sits inside `<head>` and the file still closes with `</body></html>`. Browsers auto-recover; parsers and formatters may not. Be careful running any HTML tool that rewrites these files.
- Several pages carry dead CSS for markup they don't have (`.video-background` in `projects.html`, `.resume-header` in `resume.html`, an inert `a[href^="#"]` smooth-scroll handler in `resume.html`).
- `assets/images/` holds five near-duplicate favicons; `favicon.ico` is the one actually linked.

## Content

Copy is first-person personal-brand writing in Aryan's voice — the positioning is "operator who builds." When editing prose, match the existing register (concrete, specific numbers, no marketing filler) rather than rewriting it into generic portfolio copy.
