# viveky1794.github.io

Source for [viveky1794.github.io](https://viveky1794.github.io) — my personal
site: a short bio/projects landing page plus a working knowledge base of
Linux and embedded-systems internals notes (computer architecture, memory
management, caching, scheduling, interrupts, USB, and QNX/virtualization).

## Repo structure

```
user_profile/
  index.md                  <- homepage (bio, projects, skills), permalink: /

articles/
  index.md                  <- "Articles" nav section parent
  linux/
    index.md                <- "Linux Internals" section page
    *.md                     <- one topic per file (Caching, DMA, ...)
  qnx/
    index.md                <- "QNX" section page
    *.md                     <- QNX topics, as they're written

web_backend/
  _includes/, _sass/         <- theme overrides (rarely touched)
  design/                    <- logo design deliverables, not part of the site

assets/
  images/
    logo.png, favicon.png, vivek.jpg   <- site branding
    notes/                              <- every image used inside a note,
                                            one subfolder per topic
  files/
    qnx/                    <- PDFs/epub linked from articles/qnx/

_config.yml, Gemfile        <- must stay at repo root (Jekyll/GitHub Pages
                                requirement)
sources/                    <- raw working files (excalidraw, docx), not part
                                of the site
zephyr-project/             <- empty for now
```

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
