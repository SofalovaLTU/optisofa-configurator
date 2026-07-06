# Įkėlimas į GitHub — Sofalova konfigūratorius

**Data:** 2026-07-06
**Būsena:** VISA ne-modulinė kolekcija perkonvertuota iš Optisofa šaltinio + atnaujintas apšvietimas/audinys.

## Ką įkelti

Įkelkite **VISĄ šio aplanko (`GITHUB-UPLOAD`) turinį** į repozitorijos šaknį
(`github.com/SofalovaLTU/optisofa-configurator`), perrašant senus failus.

- **147 × `.glb`** — 3D modeliai (~171 MB)
- **`index.html`** — konfigūratorius (naujas apšvietimas, audinio spalva, cache-bust `?v=…e`)
- **JSON** (`armchairs, pufai, lovos, sofos-lovos, kampines, sofos, sezlongai, fabrics, manifest`)
- **`_headers`** + **`netlify.toml`** — Netlify konfigūracija (CSP; nekeisti)
- `.md` dokumentai (neprivalomi, bet naudingi)

## Kaip įkelti dalimis (jei GitHub web ribojamas 5–6 failais)

GLB failai dideli — kelkite partijomis. **Svarbu: `index.html` įkelkite PASKUTINĮ**
(kad naujas `?v=…e` cache-bust priverstų naršyklę paimti visus naujus GLB vienu metu).

1. GLB failai (partijomis po ~10–15)
2. JSON failai (visi 9)
3. `index.html` — paskutinį

Po įkėlimo Netlify automatiškai perdiegs. Patikrinkite live svetainėje.

## Kas pasikeitė (santrauka)

- **Visi baldai perkonvertuoti** iš Optisofa 3D failų, išsaugant atskiras pagalves/kojas/piping.
- **Apšvietimas x10:** studijinis 3-taškų + minkšti HD šešėliai + kontaktinis šešėlis.
- **Audinys:** matinis, spalva 100 % = swatch nuotraukai.
- **Kojos:** teisingos (metalas juodos / ąžuolas) kiekvienam modeliui.
- **Lovos:** apmuštas pagrindas + galvūgalis + čiužinys (be grotelių).
- **Suskleista/išskleista:** kampinės, šezlongai, sofos-lovos — teisingos būsenos.
- Dublikatai pašalinti; Harris RXL → Foteliai.
