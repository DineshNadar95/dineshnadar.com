# dineshnadar.com

Personal portfolio of Dinesh Nadar — full stack developer.

The whole site is a single self-contained [index.html](index.html): plain
HTML/CSS/JS with no build step. The resume PDF and project mockup screens are
embedded directly in the file (base64 and `<template>` blocks), so there are no
other assets. The only external dependency is Google Fonts.

## Run locally

Open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server 8000
```

## Deploy

Hosted on [Vercel](https://vercel.com) as a static site (no build command,
output = repo root). Every push to `main` redeploys.

See [CLAUDE.md](CLAUDE.md) for a map of the file's structure.
