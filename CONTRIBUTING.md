# Adding content to this site

This site is built with Jekyll and a theme called Just the Docs. The theme
draws the sidebar menu on the left of every page. This guide explains how
to add new pages so they show up in the right place in that menu, and where
to put images and PDFs.

## The one rule that matters most

**The sidebar menu is built from a few lines at the top of each file (the
"front matter"), not from which folder the file sits in.** Folders just
keep the repo tidy for humans — Jekyll itself doesn't care where a file
lives. So get the front matter right, and the page will show up correctly.

Here's what a normal article's front matter looks like:

```yaml
---
layout: default
title: Interrupts
parent: Linux Internals
nav_order: 6
description: "One sentence describing this article."
---
```

## Adding a new article to an existing section (Linux Internals, QNX, or Zephyr)

1. Pick the section folder: `articles/linux/`, `articles/qnx/`, or
   `articles/zephyr/`.
2. Make a new folder for your article inside it, with an `index.md` file.
   For example, a new Linux article called "Networking" would go in
   `articles/linux/networking/index.md`.
3. Copy the front matter template from `articles/_TEMPLATE.md` and fill it
   in (see the field-by-field guide below).
4. Write the article in Markdown below the front matter.
5. If the article has images, put them in an `images/` folder right next
   to it: `articles/linux/networking/images/`. Link to them as
   `/articles/linux/networking/images/my-diagram.png`.
6. If it has a PDF or other reference file, same idea, in a `docs/`
   folder: `articles/linux/networking/docs/`.
7. Run `bundle exec jekyll serve` and check the page and sidebar before
   pushing.

**Why images live next to the article, not in one shared `assets/`
folder:** each article's pictures belong to that article. Keeping them in
the same folder means you can find, move, or delete an article and its
images together, instead of hunting through a separate folder tree to
figure out which images belong to which article.

**One thing to watch:** if an article has a lot of images, give the files
real names (`cache-line-diagram.png`) instead of the auto-generated names
a screenshot tool gives them (`1749882390239.png`). It's easier to read
the Markdown later, and it avoids two different images accidentally
getting the same name.

## Adding a whole new section (e.g. a new topic besides Linux/QNX/Zephyr)

1. Make a new folder under `articles/`, e.g. `articles/rtos/`.
2. Add `articles/rtos/index.md` with front matter like this:
   ```yaml
   ---
   layout: default
   title: RTOS
   parent: Articles
   has_children: true
   has_toc: false
   nav_order: 4
   permalink: /articles/rtos/
   description: "One sentence describing this section."
   ---
   ```
   (`nav_order` should be one higher than whatever section currently comes
   last — check `articles/linux/index.md`, `articles/qnx/index.md`, and
   `articles/zephyr/index.md` for the numbers already in use.)
3. Add articles inside it following the steps in the section above.

## Editing the homepage / profile content

The homepage (bio, skills, projects) is `user_profile/index.md`. Edit it
directly — it has `permalink: /` in its front matter, which is what makes
it load at the site's root address regardless of which folder it's in.

## Editing the site's look (theme, colors, layout)

Anything that isn't a content page — colors, fonts, layout code, logo
source files — lives under `web_backend/`. You shouldn't need to touch
this for normal writing. `_config.yml`, `Gemfile`, and `Gemfile.lock` have
to stay at the very top of the repo; that's a Jekyll/GitHub requirement,
not a choice we made.

## Front matter, field by field

| Field | What it does | When you need it |
|---|---|---|
| `title` | The text shown in the sidebar and the browser tab. | Every page. |
| `parent` | Nests this page under another page in the sidebar. Must be spelled *exactly* like the parent page's `title` (capital letters and all). | Every page except the very top-level ones (Home, Articles). |
| `nav_order` | A number that decides where this page sits among its siblings — lower numbers go first. | Every page. Check the sibling files to see which numbers are already taken. |
| `has_children` | Marks this page as a section that other pages nest under — like Articles, Linux Internals, QNX, and Zephyr. It's what makes those show an arrow in the sidebar that expands to reveal the pages inside. | Only on section pages. A plain article that nothing nests under does *not* need this. |
| `has_toc` | When a page has `has_children: true`, the theme normally also prints a plain bullet-point list of its child pages *inside the page itself*, on top of what's already in the sidebar. Setting `has_toc: false` turns that extra list off, since it just repeats the sidebar. | On every section page (paired with `has_children: true`). |
| `permalink` | Pins a page to an exact web address, instead of letting Jekyll work one out from the file's folder path. | Only on section pages (`articles/index.md`, `articles/linux/index.md`, etc.) and the homepage. A normal article doesn't need one. |
| `redirect_from` | A list of old web addresses that should forward to this page. Use it when you move or rename a page that's already live, so old links and bookmarks still work. | Only when moving a page that people might already have a link to. |
| `description` | One sentence about the page. Shows up in search results and link previews. | Every page. |

You might spot a `grand_parent` field on some older article files (it names
the section two levels up, e.g. `Articles`). You don't need to add this to
new articles — it's only needed if two different pages ever end up with
the exact same `title`, which isn't the case here.

### A quick way to remember `has_children`

Think of it like a folder icon versus a plain document icon in a file
browser. `has_children: true` says "this is a folder — other pages live
inside it." Leave it off a normal article, since a normal article is a
document, not a folder.
