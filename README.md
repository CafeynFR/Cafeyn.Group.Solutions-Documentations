# Cafeyn Group Solutions — Documentation

Public documentation for the CGS stacks, published with GitHub Pages:
https://cafeynfr.github.io/Cafeyn.Group.Solutions-Documentations/

## Structure

The documentation is in English, with one HTML file per stack.

```
index.html                  Home portal (list of stacks)
web/index.html              Web SDK documentation (including "Download the vanilla build")
backend|ios|android/        Upcoming documentation
downloads/vanilla/latest/   Latest vanilla SDK release (core/ + components/)
downloads/vanilla/vX.Y.Z/   Archived releases
```

Each documentation page is a standalone HTML file: no build step is required.

## Updating a documentation page

1. Edit `<stack>/index.html` (in Claude, an editor, or Claude Code directly in this repo).
2. Open a PR (or push to `main`).
3. GitHub Pages redeploys automatically within 1 to 2 minutes.

## Adding a new stack

1. Replace `<stack>/index.html` with the documentation.
2. In the root `index.html`, turn the stack's `<span class="soon">` line into `<a href="folder/">` and set its status to "Available".

## Publishing a new vanilla SDK release

Automatic: the `publish-to-docs.yml` workflow, installed in the SDK repo,
copies the build to `downloads/vanilla/vX.Y.Z/` and `downloads/vanilla/latest/` on every `vX.Y.Z` tag.

Manual: copy the `core/` and `components/` folders to `downloads/vanilla/vX.Y.Z/` and `downloads/vanilla/latest/`, then commit.
Remember to update the version badge and the pinned-version URL example in `web/index.html`.

Integration URLs:
- Pages: `https://cafeynfr.github.io/Cafeyn.Group.Solutions-Documentations/downloads/vanilla/latest/core/index.iife.js`
- CDN (jsDelivr, pinned version): `https://cdn.jsdelivr.net/gh/CafeynFR/Cafeyn.Group.Solutions-Documentations@main/downloads/vanilla/vX.Y.Z/core/index.iife.js`

## Reminder

This repo is **public**: never commit keys, tokens, internal URLs or customer data.
