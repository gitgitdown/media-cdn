# Structure CDN — BrandHub

## Infrastructure

```
Upload (local/script/app)
  ↓
Garage S3  →  s3://media/...          (stockage brut, port 3900)
  ↓
imgproxy   →  https://img.inthecloud.ovh   (transforms à la volée)
  ↓
Client
```

---

## Bucket `media` — Structure des objets S3

```
media/
│
├── {brand-slug}/                        ex: acme, qalyd, vhs-guru
│   ├── logos/
│   │   ├── primary/
│   │   │   ├── logo-primary-light.svg
│   │   │   ├── logo-primary-light.png
│   │   │   └── logo-primary-dark.png
│   │   ├── secondary/
│   │   │   └── logo-secondary-light.svg
│   │   ├── icon/
│   │   │   └── icon.svg
│   │   ├── wordmark/
│   │   │    └── wordmark-light.svg
│   │   └── fonts/
│   │        └── title-font.woff2
│   │        └── paragraph-font.woff2
│   │  
│   ├── fonts/
│   │   └── {family}/                    ex: inter, geist
│   │       ├── inter-regular.woff2
│   │       ├── inter-bold.woff2
│   │       └── inter.css               @font-face auto-généré
│   │
│   ├── images/
│   │   ├── og/                         Open Graph (1200x630)
│   │   ├── covers/
│   │   └── misc/
│   │
│   ├── icons/
│   │   ├── favicon.ico
│   │   ├── favicon-32.png
│   │   ├── apple-touch-icon.png
│   │   └── splash/                        icônes pour écrans de chargement
│   │        └── splash-512.png
│   │ 
│   └── brand.json                      palette, typo, guidelines
│
├── courses/
│   └── {subject}/
│       └── {topic}/                     ex: math/linear-algebra
│           ├── diagrams/
│           │   └── eigenvectors-01.webp
│           ├── screenshots/
│           ├── exercises/
│           └── covers/
│
├── userscripts/
│   └── {script-name}/
│       ├── latest/
│       │   └── script.user.js
│       ├── v1.0.0/
│       │   └── script.user.js
│       └── assets/
│           └── icon.png
│
├── websites/
│   └── {site-slug}/
│       ├── prod/
│       │   ├── css/
│       │   ├── js/
│       │   ├── fonts/
│       │   ├── images/
│       │   └── icons/
│       └── raw/
│
└── shared/
    ├── fonts/                           fonts réutilisées entre marques
    ├── icons/                           icônes communes
    └── audio/                           fichiers audio partagés
```

---

## URLs — Conventions

### Assets statiques (servis directement via nginx → Garage web endpoint)

> `https://s3.inthecloud.ovh/{brand-slug}/{chemin}`

```
https://s3.inthecloud.ovh/acme/logos/primary/logo-primary-light.svg
https://s3.inthecloud.ovh/acme/fonts/inter/inter-regular.woff2
https://s3.inthecloud.ovh/userscripts/my-script/latest/script.user.js
https://s3.inthecloud.ovh/userscripts/my-script/v1.2.0/script.user.js
```

Cache recommandé :
- `latest/` → `Cache-Control: no-cache` (URL stable, contenu variable)
- `v{x.y.z}/` → `Cache-Control: max-age=31536000, immutable`
- fonts → `Cache-Control: max-age=31536000, immutable`

### Images transformées (via imgproxy)

> `https://img.inthecloud.ovh/insecure/{transformations}/plain/s3://media/{chemin}@{format}`

```
# Resize logo 256x256 en WebP
https://img.inthecloud.ovh/insecure/rs:fit:256:256/plain/s3://media/acme/logos/icon/icon.svg@webp

# Thumbnail 300x200
https://img.inthecloud.ovh/insecure/rs:fit:300:200/plain/s3://media/acme/images/covers/hero.jpg@webp

# App icon 512x512 AVIF
https://img.inthecloud.ovh/insecure/rs:fill:512:512/plain/s3://media/acme/logos/icon/icon.svg@avif

# Open Graph 1200x630
https://img.inthecloud.ovh/insecure/rs:fill:1200:630/plain/s3://media/acme/images/og/home.jpg@jpg
```

#### Transformations imgproxy utiles

| Paramètre | Exemple | Effet |
|-----------|---------|-------|
| `rs:fit:W:H` | `rs:fit:300:200` | Resize dans la boîte, ratio conservé |
| `rs:fill:W:H` | `rs:fill:512:512` | Crop centré pour remplir exactement |
| `rs:fill:W:H/g:sm` | `rs:fill:400:400/g:sm` | Fill + smart crop (sujet au centre) |
| `@webp` | `logo.png@webp` | Conversion WebP |
| `@avif` | `logo.png@avif` | Conversion AVIF (meilleure compression) |
| `@jpg` | `image.png@jpg` | Conversion JPEG |
| `q:85` | `q:85/rs:fit:300:200` | Qualité 85% |
| `bl:5` | `bl:5/rs:fit:300:200` | Blur (ex: placeholder) |

---

## Tailles standard par usage (auto-génération à prévoir)

### App Icons
```
16x16, 32x32, 64x64, 128x128, 256x256, 512x512
```

### Social Media
```
Profile   : 400x400
Cover     : 1200x630
Story     : 1080x1920
```

### Favicons
```
favicon.ico        (16x16 + 32x32 multi-size)
favicon-32.png
apple-touch-icon.png  (180x180)
```

---

## Convention de nommage des fichiers

```
{sujet}-{variant}-{theme}.{ext}

Exemples :
  logo-primary-light.svg
  logo-primary-dark.png
  logo-icon.svg
  hero-cover-v01.webp
  banner-og-home.jpg
```

- Minuscules uniquement
- Séparateurs : tirets `-`
- Pas d'espaces, pas d'accents
- Version explicite si fichier amené à évoluer : `-v01`, `-v02`

---

## Intégrations prévues

| Outil | Rôle |
|-------|------|
| **Garage S3** | Stockage objet (source de vérité) |
| **imgproxy** | Transforms à la volée (resize, format, crop) |
| **mc (MinIO client)** | Upload CLI vers Garage |
| **App Tauri** | Interface desktop local-first, sync vers Garage |
| **Backend Fastify** | API REST, gestion métadonnées, sync |
| **Dashboard Next.js** | Admin web |
