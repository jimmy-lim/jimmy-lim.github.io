---
layout: post
title: Up and Running!
tags: web
---

- Forked jekyll-now
- Git to my computer
- Edit _config.yml in root
- Edit .md file in _posts
- Commit & Push live!!!

![_config.yml]({{ site.baseurl }}/images/first-post.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.

Update 20210603 tags added by referring to [this](https://longqian.me/2017/02/09/github-jekyll-tag)
For new tags in future
- clone a new md in tag folder
- include the tag in post's front matter section
- DONE

## Update 20260708 — modernised the site

Years-old jekyll-now, tidied up (with help from Claude Code). What changed:

**Config**
- Renamed the deprecated `gems:` key to `plugins:` in `_config.yml` (required by
  Jekyll 3+ / GitHub Pages).
- Dropped the dead Google+ footer link.
- Added a `Gemfile` (git-ignored) pinning `github-pages` so a local
  `bundle exec jekyll serve` matches GitHub's server-side build. Not needed to
  publish — this is a user site, built natively by GitHub.

**Mobile fix**
- The masthead used inline `position:absolute` with fixed % widths, which
  overrode the stylesheet's mobile rules and made the header overlap on phones.
  Removed the inline styles; `style.scss`'s `@include mobile` handles responsive
  stacking again.

**Tags, properly**
- Tag descriptions now live in one place, `_data/tags.yml` (slug → name +
  description).
- Each tag has a listing page at `/tag/<slug>/` that shows its description, and a
  new `/tags/` index lists every tag that has at least one post.
- Nav kept simple: **Blog · Tags · About**.

**Workflow**
- Added an `AGENTS.md` and a Claude Code skill (`.claude/skills/new-post/`) that
  encode the exact post format and tag wiring, so posting doesn't mean
  re-reading the whole repo each time.

New post checklist now:
- add `_posts/YYYY-M-D-Title.md` with `layout: post`, `title`, `tags`
- if it's a brand-new tag: add a `tag/<slug>.md` stub **and** an entry in
  `_data/tags.yml`
- commit & push to `master` — GitHub rebuilds automatically