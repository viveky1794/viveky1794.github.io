# Adding content

This site is built with Jekyll + the [Just the Docs](https://just-the-docs.com/)
theme. The one thing that trips people up: **where the sidebar puts a page
is controlled entirely by its front matter, not by which folder the file
lives in.** Folders are just for keeping the repo tidy for humans — Jekyll
doesn't care. So the front matter fields below are the actual contract; get
them right and the file can technically live anywhere and still show up
correctly.

## Adding an article to an existing section (Linux or QNX)

1. Copy `articles/_TEMPLATE.md` into `articles/linux/your-topic.md` (or
   `articles/qnx/your-topic.md`).
2. Fill in the front matter:
   - `title` — shows in the sidebar and browser tab.
   - `parent` — must exactly match the section page's `title`: `Linux
     Internals` or `QNX`.
   - `grand_parent: Articles` — always this, for any article one level under
     Linux/QNX.
   - `nav_order` — an integer controlling its position among siblings
     (lower = higher up). Check the other files in the same folder for the
     next free number.
   - `description` — one sentence, used in search results and social
     previews.
3. Write the article body in Markdown below the front matter.
4. If it needs images, add them under `assets/images/notes/your-topic/` and
   reference them as `/assets/images/notes/your-topic/file.png`. PDFs or
   other reference files go under `assets/files/<topic>/`.
5. `bundle exec jekyll serve` locally and check the sidebar before pushing.

## Adding a whole new section (e.g. Zephyr)

1. Create `articles/<section>/index.md` with:
   ```yaml
   ---
   layout: default
   title: <Section Name>
   parent: Articles
   has_children: true
   nav_order: <next free number under Articles>
   permalink: /articles/<section>/
   description: "..."
   ---
   ```
2. Add topic files inside `articles/<section>/` following the steps above,
   with `parent: <Section Name>` and `grand_parent: Articles`.

## Editing the homepage / profile content

Homepage content (bio, skills, projects) lives in `user_profile/index.md`.
It carries `permalink: /`, so it serves as the site root regardless of its
folder location — edit it directly, no other file needs to change. If the
profile grows a distinct new page (e.g. a full resume), add it as another
file under `user_profile/` with its own `permalink:`.

## Site code (theme, layout, config)

Anything that isn't page content — Sass overrides, include overrides, logo
design source files — lives under `web_backend/`. `_config.yml`, `Gemfile`,
and `Gemfile.lock` have to stay at the repo root; that's a hard Jekyll/
Bundler/GitHub Pages requirement, not a choice.

## Front-matter cheat sheet

| Field | Meaning |
|---|---|
| `title` | Sidebar/tab label |
| `parent` | Exact `title` of the page one level up |
| `grand_parent` | Exact `title` of the page two levels up (Just the Docs supports 3 levels max) |
| `has_children` | Set `true` on a page that has its own children |
| `nav_order` | Sort position among siblings |
| `permalink` | Only needed on section index pages (`/articles/`, `/articles/linux/`, etc.) or the homepage — regular articles don't need one |
| `redirect_from` | Old URL(s) that should redirect here (only needed when moving/renaming a page that's already been published) |
