# AGENTS.md

Personal blog for Jimmy Lim, published at <https://jimmy-lim.github.io> as a
GitHub Pages **user site**. Built natively by GitHub (server-side Jekyll, no
Actions) — push to `master` and the live site rebuilds. No plugins beyond the
GitHub Pages whitelist.

## Stack

- **Jekyll** (GitHub Pages built-in version), fork of the *Jekyll Now* template.
- Markdown posts, Liquid templates, SCSS (`style.scss` + `_sass/`).
- Plugins: `jekyll-sitemap`, `jekyll-feed` (declared under `plugins:` in
  `_config.yml`).

## Layout

| Path | Purpose |
|------|---------|
| `_posts/` | Published posts: `YYYY-M-D-Title-With-Dashes.md` |
| `_deleted/` | Draft/retired posts — **excluded from the build** (`_`-prefixed dir). Move a file here to unpublish, or to `_posts/` to publish. |
| `_layouts/` | `default`, `post`, `page`, `tagpage` |
| `_includes/` | `collecttags.html` (builds tag list for nav), meta, analytics, svg-icons |
| `tag/` | One stub per tag → generates `/tag/<slug>/` listing page |
| `images/` | Post images; reference as `{{ site.baseurl }}/images/<file>` |
| `style.scss`, `_sass/` | Styles. Mobile rules live behind the `@include mobile` mixin — do not override them with inline `position:absolute` (that caused a past mobile-overlap bug). |

## Tags → nav

The header nav (`_layouts/default.html`) **auto-lists every tag used in a post**.
Tagging a post adds it to the menu. Each tag needs a matching stub in `tag/`
(see the `new-post` skill) or its nav link 404s.

Known tags: `TOTD` (thought of the day), `Off` (non-software), `web`, `work`,
`arduino`, `DIY`, `electronic`.

## Publishing

Commit and push to `master`. GitHub Pages rebuilds automatically. There is no CI
and no build step to run locally before pushing.

## Local preview (optional)

Not required to publish. Needs Ruby headers to compile native gems:

```sh
sudo apt install ruby-dev build-essential
gem install --user-install bundler
bundle install
bundle exec jekyll serve   # http://127.0.0.1:4000/
```

`Gemfile`/`Gemfile.lock` are git-ignored (user site is built natively by GitHub).

## Adding a post

Use the **`new-post` skill** (`.claude/skills/new-post/`) — it has the exact
filename/frontmatter format and the tag-stub rule.
