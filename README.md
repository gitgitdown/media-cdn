# media-cdn

Static media assets served via GitHub + jsDelivr CDN. Logos, fonts, icons, images, and userscripts organized by brand or project.

## Features

- Brand assets (logos, fonts, icons, favicons) per slug
- Course media (diagrams, screenshots, covers)
- Userscripts with versioned and `latest` channels
- Website static assets (CSS, JS, fonts, images) per environment
- Shared resources (fonts, icons, audio)

## CDN URLs

### jsDelivr (recommended)

```
https://cdn.jsdelivr.net/gh/{user}/media-cdn@{version}/media/{path}
```

### Statically (alternative)

```
https://cdn.statically.io/gh/{user}/media-cdn/{branch}/media/{path}
```

---

## Usage examples

### Logo

```html
<img src="https://cdn.jsdelivr.net/gh/{user}/media-cdn@v1.0.0/media/acme/logos/primary/logo-primary-light.svg" alt="Acme">
```

### Font (@font-face CSS)

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/{user}/media-cdn@v1.0.0/media/acme/fonts/inter/inter.css">
```

### Favicon

```html
<link rel="icon" href="https://cdn.jsdelivr.net/gh/{user}/media-cdn@v1.0.0/media/acme/icons/favicon.ico">
```

### Userscript — versioned (stable, immutable)

```js
// @require https://cdn.jsdelivr.net/gh/{user}/media-cdn@v1.2.0/media/userscripts/my-script/v1.2.0/script.user.js
```

### Userscript — latest channel (always up to date)

```js
// @require https://cdn.statically.io/gh/{user}/media-cdn/main/media/userscripts/my-script/latest/script.user.js
```

> Use jsDelivr for versioned assets (immutable cache), Statically for `latest/` (no-cache).

---

## Cache policy

| Path pattern       | Cache-Control                           |
|--------------------|------------------------------------------|
| `latest/`          | `no-cache` — URL stable, contenu variable |
| `v{x.y.z}/`        | `max-age=31536000, immutable`            |
| `fonts/`           | `max-age=31536000, immutable`            |
| `logos/`           | `max-age=86400`                          |

---

## Repository structure

```
media/
├── {brand-slug}/              Logos, fonts, icons, images par marque
│   ├── logos/
│   │   ├── primary/
│   │   ├── secondary/
│   │   ├── icon/
│   │   ├── wordmark/
│   │   └── fonts/             Fonts liées au logo
│   ├── fonts/
│   │   └── {family}/
│   │       ├── font.woff2
│   │       └── font.css       @font-face généré
│   ├── images/
│   │   ├── og/                Open Graph 1200×630
│   │   ├── covers/
│   │   └── misc/
│   ├── icons/
│   │   ├── favicon.ico
│   │   ├── favicon-32.png
│   │   ├── apple-touch-icon.png
│   │   └── splash/
│   └── brand.json             Palette, typo, guidelines
├── courses/
│   └── {subject}/{topic}/
│       ├── diagrams/
│       ├── screenshots/
│       ├── exercises/
│       └── covers/
├── userscripts/
│   └── {script-name}/
│       ├── latest/
│       ├── v{x.y.z}/
│       └── assets/
├── websites/
│   └── {site-slug}/
│       ├── prod/              CSS, JS, fonts, images, icons
│       └── raw/
└── shared/
    ├── fonts/
    ├── icons/
    └── audio/
```

---

## Adding a new brand

1. Duplicate `media/_template/` → `media/{brand-slug}/`
2. Replace placeholder assets
3. Edit `brand.json` with palette and typography
4. Generate `@font-face` CSS (see `fonts/{family}/font.css`)
5. Tag a new version: `git tag v{x.y.z} && git push origin v{x.y.z}`

## Versioning

```bash
git tag v1.0.0
git push origin v1.0.0
```

Always tag before using versioned CDN URLs in production.
