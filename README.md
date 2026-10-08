# Yashkenia.github.io

My personal blog: exploit dev, AI bots, and Counter-Strike. Learning in public.

Live at **https://yashkenia.github.io**

It's a plain [Jekyll](https://jekyllrb.com) site with a custom layout (no theme). GitHub Pages builds it automatically on every push to `main`, so you don't need GitHub Actions or any local setup.

## Add a new post

1. Create a file in `_posts/` named `YYYY-MM-DD-your-post-title.md`, for example `_posts/2026-10-15-stack-overflows.md`.
2. Start it with front matter, then write in Markdown:

   ```markdown
   ---
   title: "Stack overflows, explained simply"
   date: 2026-10-15 20:00:00 +0530
   description: "One-line summary shown on the home page."
   ---

   Your post goes here. Use `## headings`, lists, images, and fenced code blocks.
   ```

3. Commit and push to `main` (or use **Add file → Create new file** on GitHub). The site rebuilds in a minute or two.

Notes:
- Posts dated in the future won't show up until that date.
- Reading time is calculated automatically.
- Put images in `assets/img/` and reference them as `![alt text](/assets/img/name.png)`.

## Structure

| Path | What it is |
| --- | --- |
| `_config.yml` | Site title, tagline, social links |
| `_layouts/` | Page templates (`default`, `home`, `post`, `page`) |
| `_includes/` | Header, footer, icons, reading time |
| `assets/css/main.css` | All styling (matte black, silver text) |
| `_posts/` | Blog posts |
| `about.md` | About page |

## Preview locally (optional)

```bash
bundle install
bundle exec jekyll serve
```

Then open http://localhost:4000.
