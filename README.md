# viveky1794.github.io

Source for [viveky1794.github.io](https://viveky1794.github.io) — my personal
site: a short bio/projects landing page plus a working knowledge base of
Linux and embedded-systems internals notes (computer architecture, memory
management, caching, scheduling, interrupts, USB, and QNX/virtualization).

## Repo structure

```
user_profile/
  index.md                  <- homepage (bio, projects, skills), permalink: /
  images/                    <- homepage photo, plus logo.png/favicon.png
                                (used sitewide -- see note below)

articles/
  index.md                  <- "Articles" nav section parent
  linux/
    index.md                <- "Linux Internals" section page
    caching/
      index.md               <- one article
      images/                <- images used only by this article
    dma/
      index.md
      images/
    ...                      <- one folder per article, same pattern
  qnx/
    index.md                <- "QNX" section page
    docs/                    <- PDFs/epub referenced from articles/qnx/
  zephyr/
    index.md                <- "Zephyr" section page

web_backend/
  _includes/, _sass/         <- theme overrides (rarely touched)
  design/                    <- logo design deliverables, not part of the site

_config.yml, Gemfile        <- must stay at repo root (Jekyll/GitHub Pages
                                requirement)
sources/                    <- raw working files (excalidraw, docx), not part
                                of the site
zephyr-project/             <- empty for now
```

Each article's images and reference files live in its own folder, right
next to its `index.md`. See [`CONTRIBUTING.md`](CONTRIBUTING.md) for why.

Note: `logo.png` and `favicon.png` in `user_profile/images/` are the one
exception — they're used on every page (sidebar header, browser tab), not
just the homepage. They live there because there's no other page they'd
make more sense next to, not because they're homepage-specific.

**To add a new article, or a whole new section** (e.g. Zephyr): see
[`CONTRIBUTING.md`](CONTRIBUTING.md) — it covers the front-matter contract
that controls sidebar placement, plus a copy-paste template.

## Stack

- [Jekyll](https://jekyllrb.com/) + [Just the Docs](https://just-the-docs.com/)
  (via `remote_theme`), built by GitHub Pages' standard "deploy from branch"
  pipeline — no custom Actions workflow needed.
- Content lives in plain Markdown with Just the Docs front matter
  (`title`, `parent`, `nav_order`, `description`).

## Running locally

```bash
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000`.

## Analytics

`_config.yml` has a placeholder Google Analytics 4 ID
(`google_analytics: G-XXXXXXXXXX`). To wire up real analytics:

1. Create a GA4 property at [analytics.google.com](https://analytics.google.com)
   for `viveky1794.github.io` and copy its Measurement ID (`G-XXXXXXXXXX`).
2. Replace the placeholder in `_config.yml` with that ID.
3. Optionally, also verify the site in
   [Google Search Console](https://search.google.com/search-console) to
   track search visibility.

## License

[MIT](LICENSE)
