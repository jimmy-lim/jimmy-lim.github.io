---
name: new-post
description: Publish a new blog post to this Jekyll GitHub Pages site with the correct filename, frontmatter, and tag wiring. Use whenever the user wants to add/write/publish a blog post, note, TOTD, or Off-topic entry.
---

# Publish a new blog post

This is a GitHub Pages **user site** built natively by GitHub. Publishing =
create the post file (+ tag stub if the tag is new), commit, push to `master`.
No build step, no CI.

## 1. Create the post file

Path: `_posts/YYYY-M-D-Title-With-Dashes.md`

- Date = the publish date. Month/day are **not** zero-padded (match existing:
  `2021-4-1-Hello-World.md`). Today's date if unspecified.
- Filename slug becomes part of the URL — use dashes, no spaces.

Frontmatter (exact keys):

```yaml
---
layout: post
title: <Human title, can contain spaces — NO date/timestamp prefix>
tags: <tag>                # single tag; or a list: [TOTD, off-meta-build]
description: <one line>    # optional — shown under the title on /tag/<slug>/ pages
---
```

**Do NOT put a date or timestamp in the title** (e.g. `20210401 Up and
Running!`). The post date comes from the filename and is rendered separately by
the `post` layout. Title = the human headline only.

Body is GitHub-flavored Markdown. Images go in `images/` and are referenced as
`![alt]({{ site.baseurl }}/images/<file>)`.

## 2. Tag wiring (IMPORTANT)

The nav is fixed (Blog · Tags · About). Tags are browsed via the `/tags/` index,
which auto-lists every tag that has at least one post. For a tag's page to
resolve, a stub file must exist at `tag/<slug>.md`:

```yaml
---
layout: tagpage
tag: <ExactTagAsWrittenInThePost>   # case-sensitive; must match the post's tag
permalink: /tag/<slug>/             # <slug> = lowercase, dashes for spaces
---
```

Rules:
- `tag:` value must match the post's tag **exactly** (case-sensitive) —
  `site.tags[page.tag]` is what lists the posts.
- `<slug>` in the path and permalink = the tag lowercased (e.g. tag `TOTD` →
  `tag/totd.md`, permalink `/tag/totd/`). The nav uses `slugify`, so slug must
  be the slugified tag.
- If the tag already has a stub in `tag/`, skip this step.

### Tag descriptions live in `_data/tags.yml`

Tag descriptions (shown on `/tag/<slug>/` and the `/tags/` index) are NOT in the
stub — they live in `_data/tags.yml`, keyed by slug:

```yaml
<slug>:
  name: <ExactTagAsInPost>   # case-sensitive, must match post + stub tag
  description: <one line>
```

When adding a brand-new tag, add an entry here too (else it has no description
and won't appear on the `/tags/` index).

### Existing tags (stub + data entry already present — just use them)

| Tag | Meaning | URL |
|-----|---------|-----|
| `TOTD` | Thought of the day — random thinking | `/tag/totd/` |
| `off-meta-build` | Non-software posts, life outside engineering | `/tag/off-meta-build/` |
| `web` | Web dev | `/tag/web/` |
| `work` | Work | `/tag/work/` |
| `arduino` | Arduino | `/tag/arduino/` |
| `DIY` | DIY | `/tag/diy/` |
| `electronic` | Electronics | `/tag/electronic/` |

Only create a new stub + data entry when introducing a brand-new tag.

## 3. Publish

```sh
git add -A
git commit -m "post: <title>"
git push origin master
```

GitHub Pages rebuilds automatically (~30–60s). No local build needed.

## Verify (optional)

Native local builds need `ruby-dev` (often missing). To check the live result
after push, curl the site and confirm the post + any new tag link appear:

```sh
curl -s "https://jimmy-lim.github.io/?cb=$RANDOM" | grep -oE '<a href="/tag/[^"]*">[^<]*</a>'
```

## Gotchas

- New tag but no stub → nav link 404s. Always pair a new tag with its stub.
- Draft/unpublish: put the file in `_deleted/` (excluded from build) instead of
  `_posts/`.
- Do not add inline `position:absolute` to the masthead — it breaks the mobile
  layout (`style.scss` handles responsive via `@include mobile`).
