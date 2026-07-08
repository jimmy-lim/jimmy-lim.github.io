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
| `_data/tags.yml` | Tag metadata (slug → name + description), single source for tag pages and the `/tags/` index |
| `tag/` | One stub per tag → generates `/tag/<slug>/` listing page |
| `tags.md` | `/tags/` index — all tags with descriptions + post counts |
| `images/` | Post images; reference as `{{ site.baseurl }}/images/<file>` |
| `style.scss`, `_sass/` | Styles. Mobile rules live behind the `@include mobile` mixin — do not override them with inline `position:absolute` (that caused a past mobile-overlap bug). |

## Tags

The header nav is fixed: **Blog · Tags · About**. Individual tags are NOT in the
nav — they are browsed via the `/tags/` index (`tags.md`), which lists every tag
with its description and post count. Each tag has a listing page at
`/tag/<slug>/` (a stub in `tag/`).

Known tags: `TOTD` (thought of the day), `off-meta-build` (non-software),
`web`, `work`, `arduino`, `DIY`, `electronic`. Descriptions in `_data/tags.yml`.

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
