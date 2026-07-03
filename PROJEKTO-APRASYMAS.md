# Projekto aprašymas (atnaujinta 2026-06-26)

**Projektas:** Sofalova.lt — nuosavas aukštos kokybės 3D baldų konfigūratorius, pranokstantis Optisofa. Ilgalaikis tikslas — **visas Optisofa asortimentas po Sofalova.lt vardu** viename konfigūratoriuje (bendri 75 audiniai visam asortimentui → audinių sluoksnis jau padarytas ir pakartotinai naudojamas; kiekvienam naujam produktui lieka tik 3D turinys).

**Žinių bazė:** Visada remtis `CONFIGURATOR.md` — gyvoji bazė (būsena, sprendimai, changelog). Prieš siūlant naujo, pasitikrinti, ar neprieštarauja užfiksuotiems sprendimams. Naujus sprendimus rašyti į „Atviri sprendimai" / „Changelog".

**Kaip dirbti:**
- Lietuviškai; techniniai terminai angliški, kur natūralu.
- Tiesus, sąžiningas pushback, ne pritarinėjimas. Silpna idėja — pasakyk tiesiai ir kodėl.
- Prieš didesnę užduotį — aiškinamieji klausimai; statyba tik po aiškaus patvirtinimo.
- Pamatiniai reikalavimai: tikri 3D modeliai, visi 75 audiniai su realiu spalvos keitimu, redaguojamos kainos.
- Po kiekvieno pakeitimo: sintaksės validacija (`node --check`), backup prieš keitimą, changelog įrašas.

**Techniniai faktai:**
- **Gyvas produktas:** `optisofa-configurator.netlify.app`, vienfailis `index.html` (THREE.js CDN) + `manifest.json` + `fabrics.json` + `armchairs.json`. GitHub → Netlify deploy; naudotojas kelia failus per GitHub web ir tikrina naršyklėje.
- **Veikia:** modulinė sistema (snap, presetai, Cloud 7 setai), foteliai (11 GLB, single-product režimas), audiniai (weserv proxy, box UV, per-pagalvę Cloud), kambario režimas, AR, PDF, Save&Share, kainos + mėnesinė, LT/EN/FR.
- **3D konvejeris:** FBX/OBJ → GLB per `assimp` (cm→m ×0.01); modeliai metrais. FBX UV dažnai sugadinti → box (triplanar) UV world koordinatėse generuojamas kliente.
- **3D turinys = kritinis kelias.** Funkcijų neplanuoti greičiau nei atsiranda modeliai.
- **Stack sprendimas atviras:** vienfailis neša daugiau nei planuota; Next.js migracija — tik jei atsiras reali riba (žr. sprendimą #3).
- Google Drive netinka runtime (CORS) — modeliai Netlify.
- Konversijai svarbiausia — erdvinis kontekstas („ar tilps mano kambaryje"); kambario režimas jau veikia, plėsti toliau.

**Statusas:** Gyvas MVP+, aktyviai iteruojama. 1 etapas beveik baigtas, 2 etapas (asortimento plėtra) pradėtas nuo fotelių.
