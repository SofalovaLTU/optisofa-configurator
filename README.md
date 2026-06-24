# Optisofa modulinis konfigūratorius

3D modulinių sofų konfigūratorius (Hug, Cloud, Slay, Loft).

## Struktūra
- `index.html` — konfigūratorius (Three.js, vienas failas)
- `manifest.json` — moduliai + **redaguojamos kainos** + matmenys (tiesos šaltinis)
- `fabrics.json` — 75 audiniai (spalva, nuotrauka, savybės)
- `models/{hug,cloud,slay,loft}/` — GLB 3D moduliai
- `_headers`, `netlify.toml` — CORS + cache

## Kainų keitimas
Redaguok `manifest.json` → `price_min` / `price_max` → commit. Netlify pats persideploys.

## Deploy
Prijungta prie Netlify (continuous deploy). Bet koks push į `main` automatiškai atnaujina svetainę.
