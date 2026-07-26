# Setup notes (read this first)

This is the [Satellite](https://github.com/byanko55/jekyll-theme-satellite)
theme by byanko55 (MIT licensed), customized for this site:

## What was changed from the original theme

- `_config.yml` — title/description/github_username set to yours; the
  original author's **giscus** (comments) and **goatcounter** (visitor
  counter) IDs were removed — those were wired to *his* accounts, not
  yours. Leave them blank or set up your own (see below).
- Removed the demo content that ships with the theme (`Category A`,
  `Category B`, and the four "user manual" pages) and the theme author's
  own profile photo.
- Added `_pages/Write-ups/` as a real category with one example post.

## Still needs your input

- **Photo:** replace `assets/img/avatar.svg` with a real photo, or point
  `profile_img` / `logo_img` in `_config.yml` at wherever you put it.
- **Comments (optional):** to enable them, install the giscus GitHub App
  on this repo at https://giscus.app, then fill in `giscus_repo` /
  `giscus_repoId` / `giscus_category` / `giscus_categoryId` in `_config.yml`
  with the values *that page generates for you*.
- **Visitor counter (optional):** create a free account at
  https://www.goatcounter.com and put your own code in `goatcounter_code`.

## Adding a write-up

Add a new `.md` file inside `_pages/Write-ups/` (copy
`example-writeup-template.md` as a starting point). Front matter needs
`title` and `date`; `tags` and `thumbnail` are optional. See
`_pages/Write-ups/example-writeup-template.md` for the exact format.

Want sub-categories (e.g. Active Directory, Web, Binary Exploitation)?
Create a subfolder under `_pages/Write-ups/` with its own empty
`index.md` (just two dashed lines), then drop posts inside it.

## Publish

Push this to the root of `abdallahakram20.github.io` →
**Settings → Pages → Source: Deploy from a branch → main / (root)**.
