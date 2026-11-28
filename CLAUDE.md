# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Static single-page portfolio site for an art teacher/artist (German-language content), served at `tera-art.de` (see `CNAME`; looks like GitHub Pages). There is no build step, package manager, linter, or test suite. To preview, serve the repo root over HTTP (e.g. `python -m http.server`) — opening `index.html` via `file://` won't work because galleries fetch `/images/<folder>/index.json` with absolute paths, so the site must be served from the root.

## Architecture

- **Everything lives in `index.html`**: inline CSS in `<style>`, markup for all sections (`home`, `about`, `works`, `exhibitions`, `students`, `projects`, `contact`), and several inline `<script>` blocks (smooth-scroll nav, scroll-reveal observer, lightbox, gallery loader, about-toggle, about-slideshow). Code comments are a mix of Russian and English; page copy is German.
- **Data-driven galleries**: any `<div class="gallery" folder="NAME">` is populated at load by `loadGalleries()`. It fetches `/images/NAME/index.json` (array of filenames), shuffles them, renders `.gallery-item`s from `/images/NAME/<file>`, shows the first 5 and adds a "Show more" button for the rest. Add the `show-caption` attribute to render captions.
- **Caption convention**: caption/description come from the filename — `"Title - Description.jpg"` is split on `" - "`; extension is stripped. Filenames therefore double as user-visible text.
- **Lightbox**: `setupLightboxForGalleries()` runs after galleries load and attaches click/keyboard handlers to every `.gallery`'s items; galleries added to the page get it automatically.
- **`images/index.json` files are generated** — don't hand-edit them. The GitHub Action `.github/workflows/generate-gallery.yml` runs on any push touching `images/**`, regenerates `index.json` in every image folder (jpg/jpeg/png/gif/webp only), and commits "Update gallery index.json" back to the branch. Consequently: after pushing new images, `git pull` before further local work to avoid diverging from the bot commit.

## Adding content

- New photo in an existing gallery: drop it into the matching `images/<folder>/` and push; the workflow updates `index.json`.
- New gallery: create `images/<new-folder>/`, add images, and add `<div class="gallery" folder="<new-folder>"></div>` in `index.html`.
- Folders in use: `my-works`, `projects`, `students`, `student-works`, `online-lessons`. `images/toys/` is currently not referenced in `index.html`; the top-level `images/main*.jpg(eg)` files are referenced directly by `<img>` tags (e.g. the about-section slideshow), not via the gallery mechanism.
- Note: `id="exhibitions"` is used on two `<section>` elements in `index.html` (the second is the `unterricht-section`), so `#exhibitions` nav anchors resolve to the first one only.
