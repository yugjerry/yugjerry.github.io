# Deploy guide

How to edit, preview, and publish this site.

This folder (`~/Dropbox/yugjerry.github.io/`) holds the **Jekyll source**.
The compiled HTML lives in `_site/` and is copied to a **separate** folder
(`~/yugjerry.github.io/`) that GitHub Desktop tracks and pushes to GitHub.
Visitors see `https://yugjerry.github.io` after a push.

---

## Daily workflow — preview while editing

```bash
cd ~/Dropbox/yugjerry.github.io
bundle exec jekyll serve --livereload
```

Open <http://localhost:4000>. Edit any `.md` file → save → browser auto-refreshes.

- Changes to `_config.yml` require restarting the server (Ctrl-C, then re-run).
- New files dropped in `assets/` mid-session also require a restart.

Press **Ctrl-C** when done.

---

## Publish to GitHub

Run all of this each time you want the live site to update.

### 1. Stop the dev server (Ctrl-C in the serve terminal)

### 2. Clean production build

```bash
cd ~/Dropbox/yugjerry.github.io
bundle exec jekyll clean
JEKYLL_ENV=production bundle exec jekyll build
```

### 3. Prep `_site/` for GitHub

```bash
touch _site/.nojekyll
rm -rf _site/_pages _site/requirements.txt
```

`.nojekyll` tells GitHub Pages "don't re-run Jekyll." The `rm` removes
source-y leftovers that shouldn't ship.

### 4. Sync `_site/` to the published repo

```bash
rsync -a --delete --exclude='.git' \
  ~/Dropbox/yugjerry.github.io/_site/ \
  ~/yugjerry.github.io/
touch ~/yugjerry.github.io/.nojekyll
```

(`rsync` mirrors `_site/` into the published folder, deleting anything that
no longer exists in `_site`. The `.git` folder is preserved.)

### 5. Commit & push via GitHub Desktop

1. Open **GitHub Desktop** on `~/yugjerry.github.io/`.
2. Review the diff — make sure only HTML/CSS/JS/assets show up.
3. Commit with a short message (e.g., "update news, polish about page").
4. Push to `master`.

Live site updates within ~30 seconds.

---

## Verify

After pushing, open **https://yugjerry.github.io** in an **incognito window**
(regular tabs may cache the old version).

---

## One-time setup (already done — keep for reference)

If this Mac dies and you start fresh, redo these once:

```bash
# 1. Ruby + Jekyll
brew install rbenv ruby-build
echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc
exec zsh
cd ~/Dropbox/yugjerry.github.io
rbenv install 3.2.2
rbenv local 3.2.2
gem install bundler
bundle install

# 2. ImageMagick (for responsive image thumbnails)
brew install imagemagick

# 3. Clone the published repo into a separate folder
cd ~
git clone https://github.com/yugjerry/yugjerry.github.io.git

# 4. GitHub Pages settings (one-time, on github.com):
#    Settings → Pages → Source: Deploy from branch → master → / (root)
```

---

## Common edits — where to look

| What | File |
|---|---|
| Homepage prose | `_pages/about.md` |
| News feed | `_news/announcement_*.md` (one file per item) |
| Publications | `_bibliography/papers.bib` |
| Talks | `_pages/talks.md` |
| Teaching | `_pages/teaching.md` |
| Service | `_pages/service.md` |
| Awards | `_pages/awards.md` |
| Photography page | `_pages/misc.md` (+ images in `assets/img/photography/<region>/`) |
| Site title, description, social links | `_config.yml` |
| Navbar order | `nav_order:` field in each page's front matter |
| Profile photo | `assets/img/yugui.jpg` |

---

## Adding photos to the gallery

1. Drop `.jpg`, `.jpeg`, or `.png` files into the right region folder:
   - `assets/img/photography/chicago/`
   - `assets/img/photography/china/`
   - `assets/img/photography/europe/`
   - `assets/img/photography/others/`
2. Restart `jekyll serve` so the new files get indexed.
3. ImageMagick auto-generates `-480.webp` / `-800.webp` / `-1400.webp`
   thumbnails on build. Don't add HEIC — browsers can't render it. Convert
   first:
   ```bash
   sips -s format jpeg input.HEIC --out output.jpg
   ```

---

## Common edits — where to look (URLs)

| URL | Source |
|---|---|
| `/` | `_pages/about.md` |
| `/publications/` | `_pages/publications.md` (renders `_bibliography/papers.bib`) |
| `/teaching/` | `_pages/teaching.md` |
| `/talks/` | `_pages/talks.md` |
| `/service/` | `_pages/service.md` |
| `/awards/` | `_pages/awards.md` (currently hidden from navbar) |
| `/misc/` | `_pages/misc.md` |
| `/news/` | `_news/announcement_*.md` |

---

## Troubleshooting

- **`gem install bundler` says permission denied** → `eval "$(rbenv init - zsh)"`, then retry. Long-term fix: ensure `eval "$(rbenv init - zsh)"` is in `~/.zshrc` *after* conda init.
- **`bundle install` fails on nokogiri / mini_racer** → `brew install cmake pkg-config`, retry.
- **Build hangs on `jupyter nbconvert`** → `pip install jupyter nbconvert --break-system-packages` or remove the `jekyll-jupyter-notebook` line from the Gemfile.
- **BibTeX parse error** → no `%` comments inside `@article{}` entries. `%` is not a BibTeX comment character.
- **Site shows old version after push** → check Settings → Pages source is set to `master` / `/ (root)`. Try incognito to bypass browser cache.
- **`favicon.ico not found` in serve logs** → cosmetic; set `icon: 📈` in `_config.yml` to silence.
- **Photos broken after pushing** → make sure ImageMagick variants (`-480.webp`, etc.) actually shipped. `ls _site/assets/img/photography/<region>/` should show them.

---

## What lives where (mental model)

```
~/Dropbox/yugjerry.github.io/          ← Jekyll source (edit here)
├── _pages/                              page sources (.md)
├── _news/                               news items
├── _bibliography/papers.bib             publications
├── _config.yml                          site config
├── assets/                              images, css, pdfs
├── _site/                               build output (auto-generated, do NOT edit)
└── DEPLOY.md                            this file

~/yugjerry.github.io/                  ← published repo (HTML only, pushed to GitHub)
└── (contents mirrored from _site/)
```

Never edit files in `_site/` or in the published folder directly — those are
overwritten on every build/sync.
