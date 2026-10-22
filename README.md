# Aidan Mark — classic personal site preview

This repository is the **GitHub Pages preview only**. It is not the live production website at aidanmark.dev.

[Open the preview](https://akimbo-bin.github.io/aidanmark-classic-preview/)

## What this version restores

The site uses the original ASCII name, existing personal details, and archived project imagery. Each project has its own simple HTML page and keeps its original card colours and background assets where available. The presentation is intentionally restrained: ordinary headings, old-style links, photos, tables, and long project notes. No fabricated “computer terminal” language, fake file numbers, or bracketed section IDs.

The full original Markdown for **Chroma / Inverse Synthesis, Campus Delivery Bot, Zero Trust, Heaterfy, ScheduleBee, and Sonance** is preserved in `notes/*.md` and statically rendered into the matching `.html` page. The wording and heading order are taken from `fabric-canvas:<id>.expandedMarkdown` in the original portfolio backup; the shorter introductions come from `card-text:<id>.bodyMarkdown`. Small corrections were limited to mangled text encoding and rewriting image URLs so the originals load on GitHub Pages.

**Aliens and the two client sites** did not have equivalent long project Markdown stored in that archive. They use the real titles, technical tags, supplied links and documented artwork instead of made-up descriptions. The playable game link restored from `custom-app:aliens` is https://sometimes-aliens.vercel.app/.

## Source provenance

The material was recovered from the user's local portfolio archive:

- `Documents/Portfolio/backups/localStorage-fresh-export.json`
- `Documents/Portfolio/backups/website-full-2026-04-14_011314/public/assets/`
- `Documents/Portfolio/content/`
- `Documents/Portfolio/Sometimes-Aliens-Are-Not-Your-Friends/`

The original content and images remain untouched in those locations. Images in `assets/` are local copies of the saved portfolio assets, the game, or (for Foster Media) a still from the originally saved card background video. The ScheduleBee Imgur images were saved locally so the linked demo thumbnail keeps working.

## Technical constraints

- HTML 4.01 Strict pages, CSS 2-era styling, with no JavaScript or client-side framework.
- No CSS Grid, Flexbox, animation, modern filters, gradients, custom fonts, or external style dependencies.
- GIF, JPG and PNG only for site assets. The existing newer-format reference images were converted into JPG for compatibility.
- The original large ASCII name is preserved unchanged and scrolls inside its own block on small screens.
- Sources in `notes/` remain Markdown for reference. HTML files contain the already-rendered static content; no build process runs when the page is loaded.

All design changes must remain in this independent preview repository until the owner elects to move them elsewhere.
