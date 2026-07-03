# CONFIGURATOR.md — Sofų konfigūratoriaus projektas

> Gyvoji projekto bazė ir vienintelis tiesos šaltinis. Atnaujinti kiekvieną kartą, kai priimamas sprendimas, pasikeičia apimtis ar architektūra.
> Statusas: **Funkcionalus MVP gyvas (deployed), aktyviai iteruojama**. Konfigūratorius veikia per GitHub → Netlify continuous deploy. Visos birželio 24 „atviros problemos" (audinių nuotraukos, 3D matomumas) **išspręstos**. **Cloud kolekcija — giliausiai išvystyta** (7 oficialūs setai + audinys per pagalvę).
> **Prekės ženklas: Sofalova.lt** (LT vitrina). Tikslas — **sukelti visą Optisofa asortimentą** po Sofalova vardu; visam asortimentui bendri tie patys 75 audiniai (žr. „Atviri sprendimai" #7).
> Susiję failai: **`FABRICS.md`** (75 audiniai), **`MODULES.md`** (67 moduliniai produktai), **`sofos.md`** (70 likę produktai).

---

## 1. Tikslas

Sukurti nuosavą, aukštos kokybės 3D sofų konfigūratorių pagal Optisofa pavyzdį, bet jį pranokstantį. Pamatiniai reikalavimai:

- **3D vizualizacija** su tikrais modelių failais (ne plokščios spalvos).
- **Visi audinių pavyzdžiai** ir realus spalvos/medžiagos keitimas.
- **Redaguojamos kainos** kiekvienam moduliui / produktui.
- Ilgalaikė vizija: **vienas suvienytas konfigūratorius visam Optisofa asortimentui** (Modular + Generator paradigmos) + papildomas funkcionalumas (finansavimas, atsiliepimai, lokalizacija, socialinis dalinimasis, AI asistentas, dizainerio režimas).

---

## 2. Esamų Optisofa konfigūratorių analizė

Optisofa turi **du atskirus** konfigūratorius — teisinga architektūra.

### 2a. Modulinis planuoklis (Modular)
- Atskira **Next.js + WebGL** app'ė subdomene `planner.optisofa.com` (`/eu`). Su Shopify jungiasi tik per „Add to cart".
- Šeimos: **Cloud, Slay, Loft, Hug** (keičiamos darbo metu).
- Moduliai pavadinti pagal matmenis (`SEAT_85x85`, `ARMREST_85x32`, `CUSHION_ROUND`).
- Įrankiai: View in AR, Show dimensions, Center, Change family, Clear all, Undo.
- Funkcijos: Build/Info skirtukai, gyva bendra kaina, Add to cart (rinkinys), **Save & share**, Back to Optisofa.
- Duomenų modelyje: `variants`, `color`, `colors`, `upholstery`.

### 2b. Produkto konfigūratorius (Generator) — pvz. Slay Sofabed
- Gyvena Shopify produkto puslapyje. Vienas modelis, **made-to-order**.
- Vienintelis parametras — **„Fabric finishing" su 70 audinių**. **Kaina kinta pagal audinį: €1 330–€1 650.**
- Filtrai: pagal tipą (Plain, Corduroy, Velvet, Matte Velvet, Boucle) ir spalvą (9 grupės).
- **3D su OPEN/CLOSE** — sofos→lovos transformacija (`3d_openable`).
- AR, galerija, made-to-order pristatymo data, nemokami audinių pavyzdžiai, nemokamas transportas+surinkimas, grąžinimas 14 d.
- Akordeonas: Product Details, Materials and workmanship, Basic dimensions, Delivery, Carrying and Assembly.

---

## 3. Asortimento realybė (iš katalogo)

| Rodiklis | Reikšmė |
|---|---|
| Produktų iš viso | 257 |
| Iš jų baldų | ~171 |
| Audinių (Fabric) | **75** (žr. `FABRICS.md`) |
| Produktų su 3D (`3DL`) | **tik 15** |
| Produktų su open/close (`3d_openable`) | 3 |
| Moduliniai produktai | 58 |

**Modulinės šeimos:** Slay (23), Cloud (19), Hug (12), Loft (10).
**Kategorijos:** sofos, sofos-lovos, kampinės, su šezlongu, modulinės, lovos, čiužiniai, poilsio krėslai, pufai, komodos, TV spintelės, staliukai, OUTDOOR, Outlet, priedai.

> **Kritinė išvada:** kodas nėra butelio kaklelis — **3D turinys yra**. Tik 15 iš ~171 baldo turi 3D. Pilnas padengimas = turinio gamybos projektas.

---

## 4. Techninis pamatas (PATVIRTINTA — visi modulinės šeimos sukonvertuotos)

- Optisofa 3D failai tiekiami kaip ZIP su `.obj`, `.3ds`, `.skp`, `.dwg`, `.dxf` (CAD/render, **ne web-ready**). Teisingas formatas — **šeimos aplankas su atskirais modulių poaplankiais** (kiekvienas su savo `.obj`), pvz. `Hug/Hug MCL/Hug_MCL.obj`. Atskiri modulių failai duoda švarią, lengvą geometriją.
- **OBJ → GLB konvejeris veikia** (Python `trimesh`; `merge_vertices`; decimavimas tik kraštutiniu atveju, nes **jis numuša UV**). Sunkesnius (pagalvėles ~65k) decimuoti iki ~5k.
- Geometrija švari (Blender eksportas), **realiu masteliu metrais**, su **UV** (audinį galima mapinti). **SVARBU:** Loft pufas buvo centimetrais — reikėjo ×0.01.
- **Sukonvertuota: 26 GLB (20 modulių + 6 pagalvėlės/pufai)**, visi su UV (išskyrus Loft pufą):
  - **Hug** — 6 moduliai (MCL, MCR, MSW, MSN, OTN, OTW) + 2 pagalvėlės.
  - **Cloud** — 5 moduliai (106x106, 106x85, 106x32, 85x85, 85x32) + 2 pagalvėlės.
  - **Slay** — 9 moduliai (MC, ML, MLO, MR, MRO, MS, SS, OTML, OTMR) + pufas.
  - **Loft** — **tik pufas** (atskirų modulių 3D NEBUVO įkelta; sofos failai 346–401k v — per sunkūs).
- **Trūksta / problemos:** Loft individualūs moduliai (MS/MR/ML/MCL/MCR/OTML/OTMR); Slay Armchair failas buvo tuščias (0 v). Fiksuotų produktų (`sofos.md`) GLB — dar nesukonvertuoti.
- DWG/DXF pravers vėliau dizainerio režimo eksportui (4 etapas).

---

## 5. Architektūra — `manifest.json` (vienintelis tiesos šaltinis)

Konfigūratorius skaito **vieną `manifest.json`**, ne aklai ieško failų. Tai ir yra **redaguojamų kainų sluoksnis** — kainą keiti JSON, ne kode.

Manifest sieja: produktas/modulis → GLB nuoroda + matmenys + **kaina** + leidžiami audiniai. Atskiras `fabrics` sluoksnis (iš `FABRICS.md`) bendras visiems produktams. `image` lauke esantis swatch **dvigubai naudojamas**: ir kaip mygtukas, ir kaip 3D tekstūra (per modelių UV).

Principinė struktūra (realiai — **du failai**: `manifest.json` su `families`/`products` ir atskiras `fabrics.json`; konfigūratorius juos kraunasi lygiagrečiai `init()` metu):
```
manifest.json
├─ families{}     # modulinės šeimos (Modular): {name, modules[{code,glb,dims_cm,price_min,price_max}], cushions[], note?}
└─ products[]     # fiksuoti modeliai (Generator): glb, dims, price, fabrics[], openable  (dar nepilna)

fabrics.json      # 75 audiniai: name, category, color, hex, image, comp, gsm, abrasion, features[], desc, care
```
> **Pastaba:** moduliai turi `price_min`/`price_max` (diapazonas) — kaina scenoje sumuojama iš modulių. `dims_cm` manifest'e gali būti su sukeistais W/D — Cloud atveju **tikri matmenys imami iš GLB bounding box** (`cloudLayout` matuoja modelį, ne pasitiki manifestu).

---

## 6. Hostingas — GitHub + Netlify continuous deploy (ĮDIEGTA)

**Sprendimas pasikeitė:** vietoj Netlify Drop su atskira modelių svetaine — **viena GitHub repozitorija + Netlify continuous deploy**. Konfigūratorius IR modeliai vienoje vietoje → viskas same-origin, jokio CORS.

- **GitHub repo:** `SofalovaLTU/optisofa-configurator` (šaka `main`).
- **Netlify:** prijungta prie repo, **auto-deploy nuo kiekvieno commit**. Adresas: **`optisofa-configurator.netlify.app`**. Publish dir = `.`, build command tuščias (statinė).
- **STRUKTŪRA — PLOKŠČIA (svarbu!):** GitHub web įkėlimas **neišsaugo poaplankių**, tad visi GLB atsidūrė repo **šaknyje** (`HUG_MCL.glb`, ne `models/hug/HUG_MCL.glb`). `manifest.json` keliai atitinkamai **plokšti (tik failo vardas)**. Veikia, nes vardai unikalūs.
  - Norint tvarkingos `models/` struktūros — perkelti per **`git`** (CLI/GitHub Desktop), ne web įkėlimą.
- **Atnaujinimas:** keiti failą → commit į `main` → Netlify pats persideploys. Sunkūs GLB lieka vietoje.
- **CORS:** `_headers` ir `netlify.toml` nustato `Access-Control-Allow-Origin: *` (kad audiniai/modeliai veiktų ir įdėjus į Shopify iframe).
- **Pamoka:** į GitHub kelti **išskleisto zip turinį, ne patį zip** (kitaip svetainėje tik zip → 404).

---

## 7. Asset'ų tiekimo srautas

1. Vytautas iš Optisofa Drive parsisiunčia 3D failus ir **įkelia man ZIP'ais partijomis** (po vieną šeimą/kategoriją, pvz. `Hug.zip`). Darbo aplinka išsivalo tarp sesijų — tai tik konvertavimo stalas, ne saugykla.
2. Aš: konvertuoju OBJ→GLB, išskaidau modulius, sugeneruoju/atnaujinu `manifest.json` (su kainomis ir audiniais) ir `_headers`.
3. Grąžinu vieną paruoštą `models/` aplanką.
4. Vytautas nutempia jį į Netlify modelių svetainę.
- **Audinių (swatch) imti nereikia** — juos imu pats iš Optisofa CDN.
- **Prioritetas:** bestseleriai → modulinės šeimos → fiksuoti modeliai → case goods. Produktai be 3D veikia Generator režimu su nuotraukomis.

---

## 7a. Dabartinė MVP būsena (2026-06-26)

Vienfailis `index.html` (~1065 eil., THREE.js per CDN modules) gyvai `optisofa-configurator.netlify.app`. **Abi birželio 24 atviros problemos išspręstos.**

**Render / 3D variklis:**
- WebGLRenderer su **ACESFilmicToneMapping** (exposure 0.95), **PCFSoftShadowMap** (2048), SRGB output, pixelRatio iki 2.
- **PMREM `RoomEnvironment`** kaip `scene.environment` (realistiški atspindžiai), gradientinis dangaus fonas (canvas tekstūra), grindys + grid.
- HemisphereLight + key DirectionalLight (su šešėliu, sukonfigūruota shadow kamera) + fill light.
- `OrbitControls` su damping; `pointerdown` užregistruotas **prieš** OrbitControls → mūsų select/drag handleris vykdomas pirmas (geometrinė patikra + `stopPropagation`).

**Audiniai (IŠSPRĘSTA):**
- Tekstūra kraunama per **`images.weserv.nl` proxy** (CORS-safe, leidžia nuskaityti vidutinę spalvą) su **fallback** į tiesioginį `?width=` URL. `f.image` rodo į veikiantį CDN.
- **Vidutinė spalva (`_avg`) imama iš realios nuotraukos** → tikslus base color net be tekstūros.
- **`MeshPhysicalMaterial` su sheen pagal kategoriją** (`fabricParams`): matte / velvet / boucle / corduroy / plain — kiekvienam savi roughness, sheen, bump. Procedūrinis audinio bump map.
- **Vienodas audinio tankis** per modulio dydžius: per-mesh UV `repeat` skaičiuojamas iš geometrijos bounds (T=0.5 m, clamp [0.4, 3]).

**Dėliojimas / UX (gyvai veikia):**
- **Snap tarp modulių** (`SNAP_RANGE` 0.40 m, grid 0.025 m, kraštas-prie-krašto sulygiavimas kampui/V/U arba centras tiesiai eilei), drag pele su auto-orient (atlošas/porankis automatiškai pasisuka).
- **„Sutvarkyti"** (`arrangeAll`) — sustato modulius į vieną tiesią sofą kairė→dešinė.
- **Užrakinimas** — fiksuoja sudėtį (audinį, matmenis, AR keisti vis tiek galima).
- **Pasukti / Šalinti / Centruoti / Zoom ±**.
- HQ miniatiūros (atskiras offscreen renderer) moduliams IR presetams.
- **Mobile-responsive** su skilčių perjungimu (Rinkiniai / Moduliai / Audiniai).

**Erdvinis kontekstas — „Kambarys" (dalinai 3 etapas, jau gyvas):**
- Įvedi kambario **W×D cm** → braižomos grindys + 2 permatomos sienos; **„✓ Telpa / ✗ Netelpa" badge** su likusia/trūkstama vieta. Tai ir yra didžiausias konversijos svertas iš 9 sk. — dalis jau veikia.
- **„Matmenys"** overlay — W/H/D etiketės projektuojamos į ekraną + `Box3Helper`.

**Pardavimo / dalinimosi funkcijos:**
- **Gyva kaina + mėnesinė** (`≈ €X/mėn · 12 mėn.`) — finansavimo UI elementas jau rodomas (kol kas tik vizualas).
- **AR** — surinkta sofa eksportuojama `GLTFExporter`→`<model-viewer>` (webxr / scene-viewer / quick-look, `ar-scale=fixed`).
- **PDF** (jsPDF, lazy-load) — spec lapas su scenos nuotrauka, modulių išklotine, matmenimis, audiniu, kaina + mėnesine.
- **Save & Share** — konfigūracija base64 → URL `#hash` + `localStorage` raktas **`sofalova_cfg_v2`**; atstatoma įkrovus.
- **Nemokamų audinių pavyzdžių užsakymas** (iki 10) ir **užsakymo forma** → `info@sofalova.lt`.

**Kolekcijos (presetai visoms 4 šeimoms):** Hug (tikras Optisofa receptas MCL+MSN+MCR ir kt., su modulių specifikacijos akordeonu), Slay, **Cloud (7 oficialūs setai)**, Loft.

**Cloud — giliausiai išvystyta kolekcija (žr. 7b).**

**Fiksuotų produktų kelias (`sofos.md`):** nepakitęs — `<model-viewer>` šablonas (360° + AR + audinys). Blokas tas pats: fiksuotų produktų GLB dar nesukonvertuoti (sunkūs, reikia decimavimo + UV).

Pilna **75 audinių** biblioteka su techninėmis savybėmis — žr. **`FABRICS.md`** (sudėtis %, gramatūra g/m², atsparumas 1–5, funkcinės savybės, priežiūra). `fabrics.json` su visais laukais gyvas.

---

## 7b. Cloud kolekcijos specifika (giliausiai išvystyta)

Cloud turi atskirą, sudėtingesnę logiką nei kitos šeimos:

- **`cloudLayout(r)`** — generuoja setą pagal `r.type` (M, L, P, U) arba tiesią eilę (`run`). **Tikri matmenys matuojami iš GLB bounding box**, ne iš manifesto (manifesto `dims_cm` turi sukeistus W/D).
- **7 oficialūs setai:** XS, S, M, L, XL, P, U (mygtukai „Greiti rinkiniai").
- **Porankio sulygiavimas:** porankio galas lygiuojamas su atlošo linija — `z = seat.cz − seat.d/2 − BACKD + G + ad/2`. P setui kairysis porankis prie galinės-kairės sėdynės; U setui porankiai abiejuose galuose, sparnai atviri.
- **Audinys individualiai kiekvienai pagalvei** (`rec._fab` / `rec._tex`) — pasirinkus pagalvę, audinys keičiasi tik jai.
- **Multi-row pagalvės** (`CUSH_PER_ROW=3`, `cushionRowBackStop`, `cushionRail`) — >3 pagalvės formuoja naują eilę priekyje; pagalvės clamp'inamos prie sėdynių „bėgio".
- **`addModule({raw:true})`** — apeina `normScale`, kad Cloud GLB renderintųsi tikru masteliu.
- **Serializacija praplėsta:** `fb` (audinys per modulį) + `rw` (pagalvės eilė) laukai be standartinių `c/x/z/r`.

---

## 9. Etalonas — geriausi pasaulyje (santrauka)

- **Lovesac (Threekit):** įvedi kambario matmenis → 360° virtualus kambarys; drag-drop; kiekvienas pakeitimas atnaujina matmenis IR kainą; save/share/buy.
- **SOFACOMPANY (Planner Studio):** 7 šeimos, 50 modelių, 3000+ konfigūracijų, 9 rinkos; kojų pasirinkimas; atsargų logika pagal regioną; ~9% konversija.
- **Salsita / VividWorks / iONE360:** fotorealistiškos tekstūros realiu laiku; AI asistentas natūralia kalba; presetai; taisyklės prieš negalimas kombinacijas; kambario planavimas (kelia AOV).
- **Principai:** WebGL baseline; didžiausias konversijos svertas — **erdvinis kontekstas**; AR jau standartas; grąžinimai 15–30%, priežastis „ne tai, ko tikėjausi".

---

## 10. Gap analizė — ko Optisofa neturi

| Funkcija | Optisofa | Geriausi |
|---|---|---|
| Kambario kontekstas (matmenys / scena) | ❌ | ✅ |
| Fotorealistiškos audinių tekstūros 3D | iš dalies | ✅ |
| AI dizaino asistentas | ❌ | ✅ |
| Konfigūracijos gylis (kojos, gylis, užpildas) | ❌ (tik audinys) | ✅ |
| Taisyklės prieš negalimas kombinacijas | neaišku | ✅ |
| Presetai / bestseleriai | iš dalies | ✅ |
| Finansavimas (Klarna 6/12) | ❌ | ✅ |
| Atsiliepimai prie produkto | ❌ | ✅ |
| Lokalizacija pagal šalį | ❌ | ✅ |
| Socialinis dalinimasis projektu | tik linkas | ✅ |
| Kambario set / cross-sell | ❌ | ✅ |
| Dizainerio / trade režimas + DWG eksportas | ❌ | ✅ |

---

## 11. Etapinis planas (ROADMAP)

### 0 etapas — Pamatas *(ATLIKTA)*
- ✅ OBJ→GLB konvejeris, audinių biblioteka (`FABRICS.md`), modulių/produktų duomenys (`MODULES.md`, `sofos.md`).
- ✅ Visos modulinės šeimos sukonvertuotos (26 GLB).
- ✅ Hostingas: GitHub + Netlify continuous deploy.
- ✅ Duomenų modelis (`manifest.json`); stack pradžiai (vienfailis HTML → pereisim į Next.js).

### 1 etapas — MVP: modulinis konfigūratorius *(GYVAS, didžioji dalis atlikta)*
- ✅ Vienfailis „Design Studio" (THREE.js): šeimos, moduliai, audiniai, gyva kaina, redaguojamų kainų sluoksnis (manifest).
- ✅ **Audinių nuotraukų URL** — išspręsta (weserv.nl proxy + fallback, vidutinė spalva iš nuotraukos).
- ✅ **3D modulio matomumas** — išspręsta (PMREM env, ACES tonemapping, šešėliai).
- ✅ **Snap'inimas tarp modulių**, „Sutvarkyti", auto-orient, užrakinimas.
- ✅ **AR** (GLTFExporter → model-viewer), **Save & Share** (URL hash + localStorage), **PDF** spec lapas.
- ✅ Fotorealistiškesnės medžiagos (sheen pagal audinio kategoriją, bump, vienodas UV tankis).
- ⬜ **Add to cart / Shopify** integracija (užsakymas kol kas per formą → `info@sofalova.lt`).
- ⬜ **Tikslūs Slay / Loft presetų receptai** (kode pažymėta TODO; Hug ir Cloud — tikri).
- ⬜ **Loft individualūs moduliai** (dabar tik pufas — žr. „Atviri sprendimai" #5).
- ⬜ **Peraugti į Next.js + react-three-fiber** aukščiausiai kokybei (per Claude Code) — *sprendimas atviras (#3), nes vienfailis jau neša daug funkcijų.*
- Sprendimo vartai: vizualo kokybė + UX prieš mastelį.

### 2 etapas — Katalogo mastelis
- Visos modulinės šeimos + fiksuoti apmušti modeliai.
- Taisyklių logika, presetai/bestseleriai, OUTDOOR audiniai.
- 3D failai partijomis pagal prioritetą.

### 3 etapas — Konversijos funkcijos
- Kambario kontekstas (matmenys + virtuali scena).
- Finansavimas (Klarna 6/12, mėnesinė kaina konfigūratoriuje).
- Atsiliepimai su pirkėjų kambarių nuotraukomis.
- Lokalizacija pagal šalį (EN/LT/FR + valiuta + pristatymas).
- Socialinio dalinimosi render-kortelė + bendruomenės galerija.

### 4 etapas — Intelektas ir personos
- AI dizaino asistentas (natūrali kalba → konfigūracija + audinys + mėnesinė kaina; Claude API).
- Pirkėjas ↔ Dizaineris/Trade režimas: komisiniai (Le Cercle), spec/BOM/**DWG eksportas**, kelių klientų projektai, white-label pasiūlymas.
- Cross-sell kambario rinkiniai → AOV.

### 5 etapas — Duomenų ratas ir integracijos
- Analitika apie populiarias kombinacijas → gamyba/asortimentas.
- ERP/CRM kabliukai; atsargų logika pagal regioną.
- Ilguoju — duomenys EuroTecton matchmaking'ui.

---

## 12. Atviri sprendimai

1. ✅ **Hostingas** — GitHub + Netlify continuous deploy (SPRĘSTA, žr. 6).
2. ✅ **MVP apimtis** — modulinės šeimos (Modular). Fiksuoti produktai eis per `<model-viewer>` (360+AR+audinys).
3. 🔄 **Stack** — vienfailis HTML jau neša daug funkcijų (snap, kambarys, AR, PDF, share). **Persvarstytina:** ar tikrai peraugti į Next.js + r3f, ar vienfailis pakanka iki tam tikros ribos? Sprendimas atviras — vartai: ar atsiranda lubų kokybei/UX, kurių vienfailyje nepasieksim.
4. ⬜ **Etapų eiliškumas** — ar finansavimą/lokalizaciją kelti anksčiau už 3 etapą? (Mėnesinė kaina jau rodoma; pilnas Klarna srautas — ne.)
5. ⬜ **Loft moduliai** — reikia atskirų MS/MR/ML/MCL/MCR/OTML/OTMR 3D (dabar tik pufas).
6. ⬜ **Fiksuotų produktų GLB** — kada ir kuriuos (pvz. populiariausios sofos-lovos) konvertuoti pirmus.
7. ✅ **Prekės ženklas ir vizijos apimtis** — vitrina lieka **Sofalova.lt**, bet **ilgalaikis tikslas nepakitęs: sukelti visą Optisofa asortimentą**. Pagrindas: **visam asortimentui naudojami tie patys 75 audiniai** → audinių sluoksnis (jau padarytas, `fabrics.json`) yra bendras ir pakartotinai naudojamas kiekvienam naujam produktui. Praktinė pasekmė: kiekvieno būsimo produkto „audinio pusė" jau išspręsta — lieka **tik 3D turinys** (kritinis kelias, žr. 13 sk.). Asortimentas plečiamas palaipsniui (2 etapas).
8. ⬜ **Slay / Loft presetų tikslūs receptai** — kode TODO; reikia oficialių Optisofa rinkinių sekų (kaip Hug ir Cloud).
9. ⬜ **Fotelių GLB optimizacija** — `gltfpack` (meshopt suspaudimas + kvantizacija) sumažintų ~50 MB → ~10 MB, bet reikia į `GLTFLoader` įjungti `MeshoptDecoder`. Atskiras patikrinamas žingsnis (rizika: kvantizuoti UV). Juliett II (15 MB) — prioritetas.
10. ⬜ **Fotelių spec praturtinimas** — dalis `armchairs.json` įrašų `verified:false` (matmenys iš GLB, bet savybės/aprašymas generiniai). Papildyti iš Optisofa produktų puslapių + suvesti tikras € kainas.
11. ⬜ **Likusios kategorijos** — pufai, sofos-lovos, sofos, kampinės sofos, lovos: meniu struktūra paruošta (Q2: kol kas rodom tik 2), pridedamos kai atsiranda 3D.

---

## 13. Principai / guard concepts

- **3D turinys = kritinis kelias.** Funkcijų neplanuoti greičiau nei atsiranda modeliai; produktai be 3D veikia Generator režimu.
- **Drive ≠ hostingas.** Modeliai gyvena Netlify modelių svetainėje su CORS.
- **Manifest = tiesos šaltinis.** Kainos ir failų nuorodos keičiamos JSON, ne kode.
- **MVP kodas turi būti perkeliamas** į pilną app'ę (ne aklavietė single-file).
- **Erdvinis kontekstas > produkto detalė** konversijai.

---

## 14. Darbo principai (projekto instrukcija)

- Bendrauti lietuviškai; techniniai terminai angliški, kur natūralu.
- Tiesus, sąžiningas pushback, ne pritarinėjimas. Silpną idėją pasakyti tiesiai ir kodėl.
- Prieš didesnę užduotį — aiškinamieji klausimai. **Statybą pradėti tik aiškiai patvirtinus.**
- Visada remtis šiuo failu ir `FABRICS.md`; naujus sprendimus rašyti į „Atviri sprendimai" / „Changelog".

---

## 15. Changelog

- **2026-06-23 (1)** — Sukurta bazė. Išanalizuoti abu Optisofa konfigūratoriai (live, per naršyklę). Patvirtintas OBJ→GLB konvejeris ir Hug PoC. Peržiūrėtas asortimentas (257 produktai). Parengtas etapinis planas.
- **2026-06-23 (3)** — `FABRICS.md` papildytas pilnomis techninėmis savybėmis visiems 75 audiniams (sudėtis, gramatūra, atsparumas 1–5, funkcinės savybės, priežiūra). Pataisytas ankstesnis netikslus teiginys, kad specifikacijos neskelbiamos.
- **2026-06-24 (1)** — Surinkti `MODULES.md` (67 moduliniai) ir `sofos.md` (70 likę produktai): aprašymai, matmenys, techninė info, kainos, brėžiniai (SVG), technical sheets (PDF).
- **2026-06-24 (2)** — Sukonvertuotos visos modulinės šeimos į GLB (26 failai, su UV). Loft — tik pufas (modulių nebuvo).
- **2026-06-24 (3)** — Hostingas perkeltas į **GitHub + Netlify continuous deploy** (`SofalovaLTU/optisofa-configurator` → `optisofa-configurator.netlify.app`). Struktūra plokščia (GLB šaknyje, manifest plokšti keliai).
- **2026-06-24 (4)** — Pastatytas vienfailis konfigūratoriaus MVP (`index.html`): šeimos, moduliai, 75 audiniai, gyva kaina. Gyvas. **Žinomos problemos:** audinių nuotraukų URL (timeout) + 3D modulio matomumas — žr. 7a. Kitas etapas: peraugti į Next.js + react-three-fiber.
- **2026-06-26 (1)** — **Dokumentas sinchronizuotas su gyvu `index.html` (~1065 eil.)** — buvo gerokai atsilikęs. Pokyčiai nuo birželio 24:
  - **Rebrand → Sofalova.lt** (title, PDF, `info@sofalova.lt`). Įrašytas naujas atviras klausimas (#7) dėl vizijos apimties.
  - **Abi 7a atviros problemos išspręstos:** audiniai kraunasi per `images.weserv.nl` proxy + fallback (vidutinė spalva iš nuotraukos); 3D renderinasi pilnai (PMREM `RoomEnvironment`, ACES tonemapping, PCFSoft šešėliai).
  - **Naujos gyvos funkcijos:** snap tarp modulių + „Sutvarkyti" + auto-orient, **Kambarys** (W×D → telpa/netelpa badge — dalis 3 etapo erdvinio konteksto), Matmenys overlay, užrakinimas, AR (GLTFExporter→model-viewer), PDF (jsPDF), Save&Share (URL hash + `localStorage sofalova_cfg_v2`), mėnesinė kaina, nemokamų pavyzdžių užsakymas, užsakymo forma, HQ miniatiūros, mobile-responsive.
  - **Medžiagos:** `MeshPhysicalMaterial` su sheen pagal audinio kategoriją, procedūrinis bump, vienodas UV tankis pagal geometrijos bounds.
  - **Cloud kolekcija** dokumentuota atskirai (7b): `cloudLayout` 7 setai, matmenys iš GLB bounding box, audinys per pagalvę, multi-row pagalvės, `raw:true` mastelis, serializacija su `fb`/`rw`.
  - **Duomenų modelis (5 sk.) patikslintas:** `fabrics` gyvena atskirame `fabrics.json`, ne sulieti į manifest; moduliai turi `price_min`/`price_max`.
  - **Roadmap 1 etapas** atnaujintas (dauguma ✅); stack sprendimas (#3) iš „nutarta Next.js" → „persvarstytina".
- **2026-06-26 (2)** — **Sprendimas #7 priimtas:** prekės ženklas lieka **Sofalova.lt**, ilgalaikis tikslas — **visas Optisofa asortimentas** po Sofalova vardu. Pagrindas: bendri 75 audiniai visam asortimentui → audinių sluoksnis pakartotinai naudojamas, kiekvieno naujo produkto lieka tik 3D turinys. Asortimentas plečiamas palaipsniui.
- **2026-06-26 (3)** — **2 ETAPO PRADŽIA: Foteliai (single-product režimas).** Pridėta kairės panelės **kategorijų akordeono navigacija** (kol kas 2 kategorijos: „Modulinės sofos" + „Foteliai"; sprendimas Q2). Foteliai integruoti **panaudojant esamą 3D sceną** (sprendimas Q1) — paveldi audinį, sukimą, AR, PDF, Save&Share. Detalės:
  - **11 fotelių sukonvertuota** FBX/OBJ → GLB per `assimp` (cm→m ×0.01, UV išlikę). Deploy-ready: `armchairs/fotelis-*.glb`. **Juliett II 15 MB / 539k trik. — laukia optimizacijos** (gltfpack + MeshoptDecoder, atskiras patikrinamas žingsnis).
  - **`armchairs.json`** — nauja duomenų bazė (kodas, glb, W/D/H iš GLB bounding box, kaina €, savybės, aprašymas). Kainos **apytikslės** (zł × 0.233, 2026-06) — naudotojas pataiso. Spec praturtinama iš Optisofa palaipsniui (dalis fotelių `verified:false`).
  - **`applyFabricTo` praplėsta:** prie `uvOnly:true` produktų audinys dedamas **tik ant UV turinčių mesh'ų** — kojos/rėmas lieka originalūs (modulinėms sofoms elgsena nepakitusi).
  - **Save/Share** praplėstas single režimui (`mode:'single'` + `ac:` fotelio kodas).
  - **`curMode`** ('modular' | 'single') valdo režimą; perjungimas per kategorijų antraštes.
  - **Neverifikuota naršyklėje:** (a) ar kojų/rėmo originali medžiaga atrodo gerai (jei ne — priskirti medžio/metalo medžiagą non-UV mesh'ams); (b) ar visi audinio mesh'ai turi UV.
- **2026-06-26 (4)** — **Fotelių pataisos po pirmo testo:**
  - **GLB krovimas atsparus vietai** — `loadGLB` bando kelis kelius (šaknis, `armchairs/`, `models/`); veikia nepriklausomai, kur įkelti failai. + matoma klaida, jei GLB nerastas.
  - **Kojų filtras perdarytas** — UV-buvimas pasirodė nepatikimas (kojos *turėjo* UV, korpusas ne; medžiagos visiems mesh'ams vienodos „default"). Pakeista **geometrine taisykle**: mesh'as, kurio viršus < 33% modelio aukščio = koja/pagrindas → audinys netaikomas (lieka originali pilka 0.588). Patikrinta su world-space bbox 3 modeliuose.
  - **Orientacija** — single režime kamera nukreipta beveik tiesiai į priekį (`frameAll(customDir)`; atlošas modeliuose nuosekliai -Z pusėje), vietoj numatyto 3/4 kampo, kuris foteliui atrodė pasuktas.
- **2026-06-26 (5)** — **Fotelių pataisos po antro testo:**
  - **Balta juosta po sėdyne** (apmuštas cokolis) buvo klaidingai priskirta kojoms (žema). Kojų taisyklė papildyta **pločio sąlyga**: koja = žema (viršus <34% H) **IR** plona abiem kryptim (<40% modelio pločio/gylio). Cokolis platus → gauna audinį; kojos plonos → ne. Patikrinta world-space bbox.
  - **Kojų spalvos iš Optisofa** — `armchairs.json` pridėtas `leg:{finish,color}`. Dauguma fotelių — **ąžuolo** (oak, `#CBA97B`, wood); Blues + Uma — **metalinės** (`#3C3C3E`, metal). `applyLegColor()` uždeda `MeshStandardMaterial` su tinkamu roughness/metalness. Ines/Joleen/Mia/Salma — numatyta oak (tikrinti Optisofa).
  - **Tikros kainos** (zł→€): Juliett €345 (1479 zł), Uma €478 (2050 zł), Esme €326 (1400 zł) — nebe „apytikslės".
- **2026-06-26 (6)** — **Balta apačia / Juliett II sėdynė — root cause rastas ir ištaisytas:** balta buvo ne dėl klasifikacijos, o dėl **UV trūkumo**. Cokolis ir kai kurios pagalvės (pvz. Juliett II sėdynė `Object164`) neturi UV; su tekstūra be UV `MeshPhysicalMaterial.map` negali būti nuskaitytas → mesh'as renderinasi **baltas**. Pridėta **`ensureUV()`** — mesh'ams be UV generuojamas planarinis UV (projekcija ant 2 didžiausių ašių) prieš tekstūros taikymą. Dabar visas korpusas (įsk. cokolį ir visas pagalves) gauna teisingą audinį su faktūra. Modulinėms nepaveikia (jos jau turi UV → early return).
- **2026-06-26 (7)** — **Audinio iškraipymo pataisa:** (6) planarinė UV projekcija tempė tekstūrą ant apvalių/šoninių paviršių (ypač ten, kur visas korpusas be UV, pvz. Blues). Pakeista į **box (triplanar) mapping** — kiekviena viršūnė projektuojama pagal savo normalę (YZ/XZ/XY), tankis įskaičiuotas UV koordinatėse (1 kartojimas / 0.5 m), `_rep=[1,1]`. Nebėra tempimo; ribas seka paviršiaus plokštumas natūraliai.
- **2026-06-26 (8)** — **Juliett + Juliett II — sugadintas originalus UV.** Analizė parodė, kad šių dviejų modelių originalus UV degeneruotas (kai kurių paneli U diapazonas ~0, tankis šokinėja 0.1–3.4 U/m) → tekstūra susispaudžia į dryžius. Sprendimas: `armchairs.json` pridėti `forceBoxUV:true` — `boxUV()` **perrašo** ir esamą (blogą) UV, ne tik trūkstamą (`_boxDone` cache). Pridėtas `uvHoriz:true` — box mapping'e sukeičia u/v (raštas iš kairės į dešinę, pagal pageidavimą). `ensureUV`→`boxUV(o,horiz)`.
- **2026-06-26 (9)** — **Juliett II sėdynė + spalvos tikslumas — root cause: lokalus mastelis.** `Object164` (JII sėdynė) lokalus bbox = **53×72 m** (node transform sumažina), tad `boxUV` naudojant lokalias koordinates UV tankis buvo ~100× per smulkus → tekstūra nematoma. Bendriau: skirtingi mesh'ai turi skirtingą lokalų mastelį → nevienodas, netikslus audinio atvaizdavimas. **Pataisa:** `boxUV` dabar naudoja **world koordinates** (`matrixWorld` + normal matrix) → tankis vienodas ir teisingas visiems. Box UV taikomas **visiems fotelių mesh'ams** (`legFilter`), nes FBX authored UV nepatikimi (degeneruoti diapazonai/mastelis). Rezultatas: vientisas, korektiško tankio audinys per visą fotelį.
- **2026-06-26 (10)** — **Kojų spalvos + specialūs atvejai:** Ines, Salma, Uma → **juodos metalinės** (`#3C3C3E`, kaip Blues). `classifyLegs` praplėsta: (a) **pagal pavadinimą** (`/leg|glider|foot/i`) — Uma kojos yra vienas platus mesh'as „Legs_Metal", kurio „plonas" taisyklė nepagaudavo; (b) **rėmo strypai** (plonas abiem kryptim + aukštas span>0.30) tik `frameRods:true` foteliams — Mia atlošo vertikalūs strypai (`Line042-045`) dabar juodi; feet+rėmas juodi.
