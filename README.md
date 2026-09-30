# Dr. Random

Source for [drrandom.org](https://drrandom.org). It's a [Hugo](https://gohugo.io) static site using the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, deployed to GitHub Pages by GitHub Actions on every push to `main`.

## Writing a new post

```bash
cd ~/projects/drrandom
hugo new content posts/my-post-title/index.md
```

This creates `content/posts/my-post-title/index.md` from `archetypes/posts.md`:

```yaml
---
title: "My Post Title"
date: 2026-09-30T10:59:38+01:00
slug: "my-post-title"
tags: []
draft: true
---
```

- **Write the post in Markdown** below the second `---`. Code blocks use triple backticks with a language (e.g. ```` ```rust ````).
- **Fix the title** if the auto-generated one isn't right. The folder name becomes the URL slug.
- **Tags** go in as a list, e.g. `tags: ["rust", "distributed-systems"]`.
- **`draft: true`** keeps the post out of the published site. Change it to `false` when it's ready.
- **The URL** comes from the date and slug: `/YYYY/MM/DD/my-post-title/`. This is the same pattern as the old WordPress posts (set by `[permalinks]` in `hugo.toml`).

### Images

Put images in the post's own folder and reference them relatively:

```
content/posts/my-post-title/
├── index.md
└── images/
    └── diagram.png
```

```markdown
![Alt text describing the image](images/diagram.png)
```

## Previewing locally

```bash
hugo server -D
```

Open http://localhost:1313. The `-D` flag includes drafts, and the page reloads as you save. Stop it with Ctrl+C.

## Publishing

```bash
git add content/posts/my-post-title
git commit -m "Add post: My Post Title"
git push
```

The push triggers the **Deploy to GitHub Pages** workflow (`.github/workflows/pages.yml`), and the site updates about a minute later. Check progress in the repo's **Actions** tab, or with `gh run watch`.

## Maintenance

- **Cloning fresh:** the theme is a git submodule, so clone with `git clone --recurse-submodules https://github.com/caseykramer/drrandom.git`. In an existing clone without the theme, run `git submodule update --init`.
- **Updating the theme:** `git submodule update --remote themes/PaperMod`, preview it, then commit.
- **Hugo version:** the deploy workflow pins Hugo to `0.167.0`. If you upgrade Hugo locally (`brew upgrade hugo`), bump `hugo-version` in the workflow to match.
- **Domain, DNS and HTTPS:** DNS is at DNSimple. The apex A/AAAA records point at GitHub Pages, `www` is a CNAME to `caseykramer.github.io`, and Google MX records handle email. The custom domain is set in repo Settings → Pages, with "Enforce HTTPS" on. GitHub renews the Let's Encrypt certificate automatically. If the certificate ever gets stuck, removing and re-adding the custom domain in Settings → Pages kicks it.
- **Site settings** (title, theme options, URL pattern) are in `hugo.toml`.

## Migration history

Migrated from WordPress (hosted on WinHost/IIS) in September 2026.

- 38 of 55 posts were kept, at their original URLs. Each migrated post has an explicit `url:` in its front matter to guarantee that. New posts don't need one.
- A few older posts reference images that were already lost, recorded as `<!-- missing image: ... -->` comments.
- The full WordPress export and curation notes are kept locally in `wp-export/` and `CURATION.md`, which are git-ignored and not published.
