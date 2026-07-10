# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `quarto preview` — start a live-reloading local dev server (watches `.qmd`/`.css`/`.yml` and rebuilds on save). Use `quarto preview --no-browser --port <port>` when running headless/in the background.
- `quarto render` — one-off static build to `_site/`. Run this to verify the whole site builds without errors before considering an edit done.
- No linter, test suite, or package manager — this is a static Quarto content site. `_site/` and `.quarto/` are build output; don't hand-edit them, and treat them as disposable (safe to `rm -rf` between renders).

## Architecture

This is a single-author academic website built with Quarto (`project: type: website`), not a JS framework. Content lives in `.qmd` files (Markdown + Pandoc fenced divs + optional raw HTML); Quarto compiles the whole tree per `_quarto.yml`.

**Design system**: `_brand.yml` defines the color/typography tokens (black-and-white with one navy accent, `#14213d`; Source Serif 4 headings over Inter body). `styles.css` layers custom rules on top of the `litera` Bootswatch base — both are wired in via `format.html.theme: [litera, _brand.yml]` in `_quarto.yml`. When changing visual style, prefer editing CSS custom properties at the top of `styles.css` (`--ink`, `--navy`, `--hairline`, etc.) over hardcoding colors inline.

**Reusable layout primitives** (defined in `styles.css`, used via fenced-div classes in `.qmd` files):
- `.row-list` / `.row` (with `.row-meta` + `.row-content` children) — the core two-column "faculty page" list used for CV entries, education, projects, and courses. Grid is `200px 1fr`; collapses to one column under 780px.
- `.plain-grid` — responsive multi-column grid for content with no meta/date column (e.g. Research Areas).
- `.pub-list` / `.pub-entry` — citation-style publication entries (`.pub-year` + `.pub-body`, with `.pub-self` bolding the site owner's name in author lists).
- `.hero`, `.hero-figure`, `.btn-line` (`.btn-solid`/`.btn-outline`) — homepage hero section and CTA buttons.
- `.split` / `.split-narrow` — generic two-column page layout (About, Contact).

Pandoc fenced divs (`::: {.class}` ... `:::`) must have the opening/closing fence on their own line — inline forms like `::: {.class}text:::` fail to parse. Keep this in mind when editing rows.

**Single-source content via includes**: `_includes/featured-projects.qmd` and `_includes/recent-publications.qmd` are partials pulled into multiple pages with `{{< include _includes/... >}}` (e.g. `index.qmd`, `projects.qmd`, `publications.qmd`, `cv.qmd` all reuse the same source instead of duplicating entries). When updating "recent" or "featured" items, edit the partial once rather than each page. Files under `_includes/` are not standalone pages — they're excluded from the render glob in `_quarto.yml` (`project.render: ["*.qmd", "news/"]` only matches root `.qmd` files and `news/`).

**News is a Quarto listing, not a hand-maintained page**: each update is its own dated file in `news/YYYY-MM-DD-slug.qmd` with `title`/`date` frontmatter. `news.qmd` renders the full chronological listing (`listing: contents: news`). `index.qmd` renders a second, independent listing (`id: home-news`, `max-items: 3`, `type: table`) pointing at the same `news/` directory — dropping a new file there updates both pages automatically with no manual sync.

**Placeholder assets**: `images/` and `files/` hold only `README.md` placeholders (no real headshot/CV yet). `index.qmd`'s `.hero-figure` div is a CSS-only grayscale placeholder with a code comment showing how to swap in a real `<img src="images/profile.jpg">`. `cv.qmd`'s download button and the hero's "Download CV" button both point at `files/Usman_Ahmed_CV.pdf`, which doesn't exist yet.

**Bracketed placeholder text**: content throughout (institution name, advisor, dissertation title, award names, dates, social links) uses `[Bracketed]` placeholders for the site owner to fill in — these are intentional, not TODOs to silently invent values for.
