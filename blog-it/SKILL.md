---
name: blog-it
description: "Write and publish a post to a personal Jekyll + GitHub Pages blog — use when the user says \"blog this\", \"write a post\", \"push to the blog\", or \"/blog\""
---

## Blog — Jekyll + GitHub Pages

**Repo:** `~/repos/github.com/<you>/<you>.github.io`
**Engine:** Jekyll + GitHub Pages, theme: minima
**Branch:** `master` → auto-deploys on push
**Index:** keep a blog index notes file (e.g. `~/notes/blog.md`) — update after every post

**Note** — No local Jekyll build needed. GitHub builds on the server side.

---

## ⚠️ Hard rule: no employer information

NEVER include in any post:
- Your employer's internal tools, systems, codenames, or infrastructure
- Internal project names, team names, org structure, or roadmap details
- Customer names, partner names, or contract details
- Internal tickets, wiki pages, or meeting content
- Salary, headcount, budget, or business metrics

Write about **techniques, tools, and patterns** — not where you work or what your employer ships. When in doubt, ask the user before publishing.

---

## Workflow

### 1. Write the post

```
_posts/YYYY-MM-DD-slug-here.md
```

Frontmatter:
```yaml
---
layout: post
title: "Your Title Here"
date: YYYY-MM-DD
categories: ai agents productivity   # space-separated
series: "AI coding agent productivity"   # omit if not in a series
---
```

### 2. Style

- **Lead with the problem**, not the solution
- One concrete opener sentence — no "In this post I will..."
- Code blocks for anything runnable
- Tables for comparisons
- End with a `## Source` section linking repos/references
- No LLM fluff — write like a README
- Escape `@word` as `\@word` to avoid typst citation errors if PDF ever needed

Pick one well-regarded published post on the blog and use it as the reference for tone and density.

### 3. Verify every link BEFORE publishing (do not skip)

Citing URLs from memory produces 404s. Check each link's HTTP status first:

```bash
for u in "https://example.com/a" "https://example.com/b"; do
  echo "$(curl -sS -L -o /dev/null -w '%{http_code}' "$u")  $u"
done
```

Every link MUST return `200` (or `301/302` to a valid target) before commit. This is cheaper than shipping a dead link and fixing it after the user catches it.

### 4. Commit and push

```bash
cd ~/repos/github.com/<you>/<you>.github.io
git add _posts/YYYY-MM-DD-slug.md
git commit -m "post: <title>"
git push   # GitHub Pages deploys automatically in ~60s
```

### 5. Update the index

Add a row to the `## Posts` table in the blog index notes file (newest first), and append the post to the `## Active series` row if it belongs to a series:

```
| YYYY-MM-DD | Title | `category/path/YYYY/MM/DD/slug.html` |
```

The URL follows Jekyll's `permalink` pattern derived from `categories` + `date` + slug.

### 6. Verify it actually went live (do not skip)

A **successful Pages build does NOT mean the post is live** — Jekyll silently drops
future-dated posts. Curl the real URL after the build completes:

```bash
base="https://<you>.github.io"; slug="/CATEGORY/PATH/YYYY/MM/DD/slug.html"
curl -sS -L -o /dev/null -w "%{http_code}\n" "$base$slug"   # must be 200
curl -sS -L "$base/" | grep -o 'slug[^"]*html' | head -1     # must appear on homepage
```

If it 404s but the build succeeded, it's almost certainly the future-date trap below.

---

## Published-post update safeguards

- Preserve the original narrative and append a clearly labeled update unless the user explicitly requests a rewrite.
- HTTP-check only newly added or modified URLs; do not repeatedly check unchanged published links unless a link audit is requested.
- After pushing, verify the real published URL returns HTTP 200.

---

## Tips

- Keep a `## TODO` section in the blog index notes file for draft ideas; check it before starting from scratch
- If the post continues or contradicts a prior post, link it with `{% post_url YYYY-MM-DD-slug %}`
- **Pre-dawn publishing + future-date trap:** Jekyll interprets a dateless
  `date: YYYY-MM-DD` as **midnight in the site timezone** and **silently drops posts dated
  in the future**. Before `_config.yml` had an explicit `timezone:` line, a post written at
  04:00 local time was read as midnight UTC = ~1h in the future → dropped until UTC caught
  up. Set `timezone:` in `_config.yml` and do NOT remove it. If a pre-dawn post still 404s
  after a successful build, add an explicit time to the frontmatter
  (`date: YYYY-MM-DD 04:00:00 +0530`).
