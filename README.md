# Cafeyn Group Solutions — Documentation

Public documentation for the CGS stacks, published with GitHub Pages:
https://cafeynfr.github.io/Cafeyn.Group.Solutions-Documentations/

## Structure

The documentation is available in French (`fr/`) and English (`en/`), with one HTML file per language and per stack.

```
index.html                  Redirects to fr/ or en/ (saved language, otherwise the browser's)
fr/index.html               Home portal (list of stacks), in French
fr/web/index.html           Web SDK documentation, in French (including "Download the vanilla build")
fr/backend|ios|android/     Upcoming documentation, in French
en/…                        Same tree, in English
downloads/vanilla/latest/   Latest vanilla SDK release (core/ + components/)
downloads/vanilla/vX.Y.Z/   Archived releases
```

Each documentation page is a standalone HTML file: no build step is required.

## Languages

- Every page has an FR / EN switcher that leads to the same page (and the same anchor) in the other language, and remembers the choice (`localStorage`, key `cgs-docs-lang`).
- Pages declare their language (`<html lang>`) and their translations (`<link rel="alternate" hreflang>`).
- **Any change to a page must be applied to the other language in the same PR.** Element `id`s and anchors must stay identical between `fr/` and `en/`.
- To add a language: copy `en/` to `<code>/`, translate it, then add the language to the switcher, the `hreflang` tags and the redirect script of the root page.

## Updating a documentation page

1. Edit `fr/<stack>/index.html` **and** `en/<stack>/index.html` (in Claude, an editor, or Claude Code directly in this repo).
2. Open a PR (or push to `main`).
3. GitHub Pages redeploys automatically within 1 to 2 minutes.

## Adding a new stack

1. Replace `fr/<stack>/index.html` and `en/<stack>/index.html` with the documentation.
2. In `fr/index.html` and `en/index.html`, turn the stack's `<span class="soon">` line into `<a href="folder/">` and set its status to "Disponible" / "Available".

## Publishing a new vanilla SDK release

Automatic: the `publish-to-docs.yml` workflow, installed in the SDK repo,
copies the build to `downloads/vanilla/vX.Y.Z/` and `downloads/vanilla/latest/` on every `vX.Y.Z` tag.

Manual: copy the `core/` and `components/` folders to `downloads/vanilla/vX.Y.Z/` and `downloads/vanilla/latest/`, then commit.
Remember to update the version badge and the pinned-version URL example in `fr/web/index.html` and `en/web/index.html`.

Integration URLs:
- Pages: `https://cafeynfr.github.io/Cafeyn.Group.Solutions-Documentations/downloads/vanilla/latest/core/index.iife.js`
- CDN (jsDelivr, pinned version): `https://cdn.jsdelivr.net/gh/CafeynFR/Cafeyn.Group.Solutions-Documentations@main/downloads/vanilla/vX.Y.Z/core/index.iife.js`

## Reminder

This repo is **public**: never commit keys, tokens, internal URLs or customer data.
