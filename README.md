# viveky1794.github.io

Source for [viveky1794.github.io](https://viveky1794.github.io) — my personal
site: a short bio/projects landing page plus a working knowledge base of
Linux and embedded-systems internals notes (computer architecture, memory
management, caching, scheduling, interrupts, USB, and QNX/virtualization).

## Repo structure

```
index.md                    <- homepage (bio, projects, skills)
linux-internals.md          <- "Linux Internals" nav section parent
linux-internals/*.md        <- one topic per file (Caching, DMA, ...)
qnx.md                      <- "QNX" section (single page for now)

assets/
  images/
    logo.png, favicon.png, vivek.jpg   <- site branding
    notes/                              <- every image used inside a note,
                                            one subfolder per topic
  files/
    qnx/                    <- PDFs/epub linked from qnx.md

_config.yml, _includes/, _sass/    <- theme config/overrides (rarely touched)
design/                     <- logo design deliverables, not part of the site
sources/                    <- raw working files (excalidraw, docx), not part
                                of the site either
zephyr-project/             <- empty for now, see below
```

**To add a new note to an existing section** (e.g. another Linux Internals
topic): create `linux-internals/your-topic.md` with front matter matching
the other files there (`layout: default`, `title`, `parent: Linux
Internals`, `nav_order`, `description`), and drop any images it needs in a
new `assets/images/notes/your-topic/` folder, referenced as
`/assets/images/notes/your-topic/file.png`.

**To add a whole new top-level section** (e.g. Zephyr, once there's
content): follow the QNX pattern — a single `zephyr.md` at the repo root is
enough to start (it doesn't need its own folder until it grows past one
page, at which point copy the `linux-internals.md` + `linux-internals/`
pattern). Give it `nav_order` after the existing sections in
`_config.yml`-adjacent front matter.

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
