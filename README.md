# Tobias's Blog

A [Hugo](https://gohugo.io/) blog using the [Paper](https://github.com/nanxiaobei/hugo-paper) theme,
deployed to GitHub Pages at <https://bertobi.github.io/blog/>.

Posts live in `content/posts/` as Markdown files with YAML front matter:

```markdown
---
title: "Post Title"
date: 2026-06-13
tags: []
draft: false
---

Post body in Markdown…
```

## Deploys

The GitHub Actions workflow (`.github/workflows/deploy.yml`) builds the site and
publishes it to GitHub Pages **on every push to the `master` branch** (the default
branch, and the only branch the `github-pages` environment permits to deploy).
Posts, the tool, and the workflow all live on `master`.

## Blog Publisher (`blog-publisher.html`)

A self-contained writing app for creating and publishing posts without touching
the Git CLI. It's a single HTML file with no dependencies — open it directly in a
browser (double-click, or `file://…/blog-publisher.html`). It works on any machine
where you've cloned this repo.

### First-time setup

1. Open `blog-publisher.html`.
2. When prompted, paste a **GitHub personal access token** with repo write access:
   - Classic token with the `repo` scope — generate one at
     <https://github.com/settings/tokens/new?scopes=repo&description=Blog%20Publisher>, or
   - Fine-grained token scoped to the `blog` repo with **Contents: Read and write**.
3. (Optional) Use **Test connection** in Settings to verify access.

The token is stored only in that browser's `localStorage` and is sent only to
GitHub — it's never committed. You'll re-enter it once per device.

### Writing & publishing

- Write in the Markdown editor; the preview pane updates live.
- Fill in **Title**, **Date**, and optional **Tags**. The filename is generated
  automatically as `YYYY-MM-DD-slug.md`.
- Hit **Publish** (or `Ctrl/Cmd+Enter`). The app commits the file to
  `content/posts/` via the GitHub Contents API **on the `master` branch**, which
  triggers the deploy. The post is live in ~1–2 minutes.

### Features

- **Drafts** — work is auto-saved to `localStorage`; close and reopen anytime.
  The sidebar lists your drafts.
- **Stage as draft** — check this to commit the post with `draft: true`. It lands
  in the repo (and syncs across machines) but stays hidden from the live site
  until you uncheck it and re-publish.
- **Manage posts** — the **Posts** tab lists everything in `content/posts/`; click
  to edit (changes commit back to the same file, preserving the URL) or delete.
- **Images** — the 🖼 button (or paste / drag-drop into the editor) uploads an
  image to `static/images/` and inserts the Markdown reference.
- **Stats** — the **Stats** tab shows total and per-post view counts (see below).
- **Light/dark theme** toggle (`☾`/`☀`) and **editor↔preview scroll sync** (`⇅`).
- **Shortcuts** — `Ctrl/Cmd+B` bold, `Ctrl/Cmd+I` italic, `Ctrl/Cmd+K` link,
  `Ctrl/Cmd+S` save draft, `Ctrl/Cmd+Enter` publish.

> **Note:** the live preview uses a built-in Markdown renderer — it's a close
> approximation of Hugo's output, not an exact match.

## Analytics (GoatCounter)

Views are tracked with [GoatCounter](https://www.goatcounter.com) — a free,
privacy-friendly, cookie-less analytics service (no consent banner needed).

**Setup (one time):**

1. Create a free site at <https://www.goatcounter.com> — pick a code, e.g.
   `tobiasblog` (your dashboard becomes `tobiasblog.goatcounter.com`).
2. Set that code in `hugo.toml` under `[params]`:
   ```toml
   [params]
     goatcounter = 'tobiasblog'
   ```
   This injects the tracking snippet site-wide (via `layouts/partials/footer.html`).
   Leave it empty to disable. Commit + deploy, and views start counting.
3. To see stats **inside the Blog Publisher tool**, also enter the same code in
   the tool's **Settings → GoatCounter code**, and in GoatCounter enable
   **Settings → "Allow adding visitor counts on your website"** (off by default).
   The **Stats** tab then shows total + per-post views, pulled from GoatCounter's
   public counter endpoint.

Full analytics (referrers, browsers, time series) live in your GoatCounter
dashboard. The tool's Stats tab is a quick at-a-glance view counter.
