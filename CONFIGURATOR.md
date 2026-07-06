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
- **2026-06-26 (12)** — **Fotelių spec iš Optisofa (`optisofa.com/en-de`, EUR):** `armchairs.json` praturtintas pilnu spec. **Tikros EUR kainos** (en-de puslapis) — labai pasikeitė nuo mano zł įverčių: Mia €680 (buvo €279!), Blues €425, Ines €500, Uma €490, Salma €470, Emilly €370, Esme €440, Harris €420, Joleen €420, Juliett €480, Juliett II €475. Pridėti: matmenys (P/G/A, sėdynės aukštis/gylis, svoris, prošvaisa), medžiagos (sėdynės/atlošo užpildas, rėmas), nuimamas užvalkalas, kietumas, 120 kg, 5 m. garantija. Kojos patvirtintos: Blues+Ines **juodas plienas**, Emilly/Esme/Harris/Joleen/Juliett/Juliett II **vaškuotas ąžuolas**, Uma/Mia **metalas**, Salma **mediena**. `buildArmSpec` praplėstas rodyti visą spec. **6 pilnai patikrinti** (Blues, Emilly, Esme, Harris, Ines, Joleen); **5 (Mia, Salma, Juliett, Juliett II, Uma)** — kaina+kojos iš Optisofa, matmenys iš 3D modelio, pilnas Optisofa spec dar netikrintas (`verified:false`, Optisofa puslapiai labai dideli — užbaigsim kitą kartą).
- **2026-06-26 (11)** — **Fotelių UI + i18n:**
  - **Miniatiūros iš priekio** — `thumbFor(glb,front)`; foteliams `front=true` (`renderThumb` priekinis kampas), nebe 3/4 (atrodė pasukti).
  - **Aprašymas po kortelėmis** — `buildArmSpec` auto-`scrollIntoView` pasirinkus fotelį (matomas kaip audinio spec).
  - **Didesnis kategorijų mygtukas** — `.cat-hd` padidintas (16.5px, soft fonas, hover, brass chevron 20px, aktyvaus rėmelis).
  - **Kalbos LT/EN/FR** — perjungiklis header'yje kairėje nuo kainos. `data-i18n` sistema (23 statiniai UI tekstai) + `t()` dinaminiams (hint, kaina, užrakinimas, filtrai, fotelio spec). Kalba saugoma `localStorage` (`sofalova_lang`). **Neišversta (kol kas):** modalų vidus, fotelių/audinių aprašymai `armchairs.json`/`fabrics.json` (produktų turinys) — kitas žingsnis.
- **2026-06-26 (13)** — **Meniu ir kameros patikslinimai:** (a) `openCat(cat,force)` — **toggle** elgsena: paspaudus ant atviros kategorijos ji susiskleidžia; `force=true` programiniam atidarymui (restoreConfig). (b) Kategorijos pavadinimas **iscentruotas**, chevron absoliučiai dešinėje. (c) Fotelio kamera — **3/4 hero kampas akių lygyje** (`frameAll(0.55,0.32,1.0)`), pagal naudotojo patvirtintą etaloną (Blues ekrano nuotrauka). (d) Naudotojo projekto aprašymas (project instructions) sinchronizuotas su realybe — pateikta atnaujinta versija.
- **2026-06-26 (14)** — **Fotelių sąrašo UI + visi 11 verified:** (a) miniatiūros — 3/4 hero kampas (kaip pagrindinis vaizdas) + didesnis atstumas (dist×2.15), fotelis nebeišlenda iš rėmelio; (b) kortelės pavadinimas ir kaina **iscentruoti**, matmenys iš kortelės pašalinti; (c) **spec įterpiamas po paspaustos kortelės eilute** (grid `1/-1`, `rowEnd.after(spec)`), ne sąrašo gale; (d) likusių 5 fotelių Optisofa spec: Mia (sėdynė 42/50, W75, plieno rėmas, boutique — €680 pagrįsta!), Salma (**plieninės kojos** — pataisyta iš medžio!), Juliett (50s, išlenkti porankiai), Juliett II (kolekcijos desc), Uma (HR35, metalinė konstrukcija). **Visi 11 verified:true**.
- **2026-06-26 (15)** — **Konversijos auditas** (`KONVERSIJOS-AUDITAS.md`): pilnas vertinimas pirkėjo piltuvėlio perspektyva. Pagrindinė išvada: konversiją riboja ne 3D funkcijos, o **pirkimo užbaigimo ir pasitikėjimo sluoksnis** (checkout, pristatymo/grąžinimo info, produkto nuotraukos, mobile, analitika). Rekomenduota seka: pasitikėjimo blokas + nuotraukos + GA4 (mažos pastangos / didelis poveikis), lygiagrečiai — checkout sprendimas (Shopify vs Montonio/Stripe). Nauji atviri klausimai: #12 checkout platforma, #13 audinių kainų grupės, #14 kojų pasirinkimo UI, #15 finansavimo partneris (Inbank/ESTO).
- **2026-06-26 (16)** — **Saugumo auditas (vienkartinis; „24h loop" neįmanomas — Claude neveikia fone; nuolatiniam stebėjimui siūlyta UptimeRobot/Netlify monitoring):** API endpointų NĖRA — statinis puslapis (index.html + JSON + GLB), nėra backend/DB/raktų. Patikrinta: (1) užsakymo forma → `mailto:` (duomenys niekur į serverį nekeliauja, tik į naudotojo pašto programą); (2) URL hash (share nuorodos = svetimi duomenys) — visos reikšmės tikrinamos prieš mūsų sąrašus (`FABRICS.find`, `MANIFEST.families`, modulių kodai), į `innerHTML` svetimi duomenys nepatenka → XSS per hash nėra; (3) naudotojo įvestis (formos, roomW/D) nepatenka į innerHTML, roomW/D per `parseFloat`; (4) CDN versijos prisegtos (`three@0.160.0`), bet be SRI — priimtina rizika, ilgainiui self-host; (5) **trūko saugumo antraščių** → sukurtas `_headers` (X-Frame-Options DENY, nosniff, Referrer-Policy, Permissions-Policy, CSP pagal faktinį domenų sąrašą: jsdelivr, cdnjs, weserv, optisofa, fonts). CSP su 'unsafe-inline' (vienfailio inline script neišvengiamybė) — riboto griežtumo, bet blokuoja svetimus script domenus. Deploy: `_headers` į repo šaknį; jei kas sulūžta — ištrinti failą (trivialus rollback).
- **2026-06-26 (17)** — **SPRENDIMAS #12 PRIIMTAS: Shopify, 1 lygis (krepšelio permalink).** Parduotuvė jau veikia Shopify; konfigūratorius lieka Netlify kaip įrankis, bet „Užsakyti" single režime veda tiesiai į Shopify krepšelį. Implementuota: (a) `SHOPIFY_DOMAIN` konstanta + `shopifyCartURL()` — `GET /cart/add?id=<variant>&properties[Audinys]=...&properties[Kojos]=...&properties[_Konfigūracija]=<share URL>` (line item properties; `_` prefiksas slepia nuo pirkėjo, matoma užsakyme); (b) `armchairs.json` — `shopify_variant:null` laukai + instrukcija kaip rasti varianto ID (Shopify Admin → Products → variantas → ID iš URL); (c) **fallback**: kol domenas/ID neįrašyti — sena užsakymo forma, niekas nesulūžta; (d) **gilioji nuoroda Į konfigūratorių**: `?p=fotelis-blues` — parduotuvės produkto puslapio mygtukas „Konfigūruoti 3D" atidaro su pasirinktu foteliu (seka: hash → ?p= → localStorage). Pastaba: GET /cart/add su properties — ilgametis stabilus Shopify elgesys, bet nedokumentuotas kaip API; jei kada nustotų veikti, atsarginis kelias — cart permalink + note. Moduliniai (sofos) kol kas per formą — jiems Shopify produktų dar nėra.
- **2026-06-26 (18)** — **Shopify produktų CSV** (`shopify-foteliai.csv`): 11 fotelių sugeneruota iš `armchairs.json` — pavadinimai, HTML aprašymai (desc + savybės + spec lentelė), kainos EUR, SKU (FOT-*), svoris, made-to-order (be inventoriaus sekimo), handle = mūsų kodai (`fotelis-blues` — sutampa su giliosios nuorodos `?p=` formatu). Importas: Shopify Admin → Products → Import. Po importo: variantų ID paimami iš `https://<domenas>/products/<handle>.json` (laukas `variants[0].id`) ir surašomi į `armchairs.json` `shopify_variant`. Nuotraukos į CSV neįtrauktos — kelti ranka arba iš Optisofa dilerio medžiagos (teisės — per tiekėjo santykį). SVARBU: kainos vientisumas — nuo šiol Shopify tampa kainos šaltiniu; keičiant kainą Shopify būtina atnaujinti ir armchairs.json.
- **2026-06-26 (19)** — **Spec kortelės akordeonas + kompaktiška įrankių juosta:** (a) fotelio spec kortelė pasirinkus rodo tik **pavadinimą, kainą (+mėnesinę) ir matmenis**; žemiau — „Aprašymas +" mygtukas (i18n: `descWord` LT/EN/FR), kurį paspaudus išsiskleidžia visa likusi informacija (sėdynė, kojos, savybės, aprašymas, techninė), „+" tampa „−", paspaudus „−" susiskleidžia atgal; (b) apatinė įrankių juosta sumažinta (mygtukai 6×8px / 11px šriftas, tarpai 0, padding 4px) + **permatomas fonas su backdrop blur** — net persidengdama nebeuždengia fotelio, nuleista prie 12px nuo apačios.
- **2026-07-05 (1)** — **Audito pataisos + naršyklės verifikacija (repo klonas, lokalus http serveris + visi GLB/JSON).** Padaryta ir realiai patikrinta naršyklėje:
  - **`index.html` (3):** (a) **audinių nuotraukų URL** — pridėta `fimg(url,w)`, kuri teisingai prijungia `width` su `?`/`&`; pataisyti 3 iškvietimai (renderSwatches, om swatch img, om detail). *Patikslinimas: gyvas `fabrics.json` turi **75** įrašus ir **visi** `image` jau su `?`, tad tai **apsauginė** pataisa (ateities OUTDOOR audiniams be `?` + logika suvienodinta su `loadFabricTexture`), ne gyvo bug'o taisymas. Ankstesnis „120 įrašų / dalis be ?" buvo WebFetch klaida.* (b) **`boxUV` bendros geometrijos apsauga** — prieš perrašant world-space UV, `o.geometry=o.geometry.clone()`, kad kelių to paties GLB egzempliorių (klonai dalijasi cache geometriją) UV nesikorumpuotų. (c) **pinch-zoom a11y** — pašalintas `maximum-scale=1`.
  - **`armchairs.json` — realus radinys:** `glb` keliai buvo `armchairs/fotelis-*.glb`, bet failai guli repo **šaknyje** (nėra `armchairs/` aplanko). `loadGLB` fallback tai išgelbėdavo, bet **kiekvienas fotelis pirma gaudavo 404, po to sėkmę** (bereikalingas round-trip + triukšmas, ir gyvame Netlify). Keliai pataisyti į šakninius (`fotelis-*.glb`) → įkėlimas iš karto 200 (patvirtinta: `HEAD fotelis-blues.glb` → 200, jokių naujų 404).
  - **Verifikuota naršyklėje:** modulinė sofa (Hug) įkelia + audinys dedasi; 75 audinių swatch'ai rodo realias nuotraukas; foteliai įkelia iš šaknies (Blues — korpusas su audiniu + juodos metalinės kojos); konsolė be klaidų.
  - **Neliesta (reikia tavo sprendimo):** mėnesinės kainos rodymas be realaus finansavimo (verslo/atitikties klausimas); pilnas i18n modalams/PDF/spec etiketėms; negyvas/dubliuotas mobilus CSS.
- **2026-07-05 (2)** — **Boho demo (`configurateur-boho-DEMO.html`) — kritinė klaida IŠTAISYTA.** `bodyMats` buvo naudojamas (`if(bodyMats.indexOf(...))` ties body mesh'u `addModule` viduje), bet **niekur nedeklaruotas** → `ReferenceError: bodyMats is not defined` → `traverse` nutrūkdavo, `scene.add(root)` neįvykdavo, pirmas modulis neužsikraudavo, „Préparation…" pakibdavo. Pataisa: prie deklaracijų pridėta `const bodyMats=[];`. **Verifikuota naršyklėje:** demo dabar įkelia 3D modulį ir generuoja miniatiūras, konsolė be `bodyMats` klaidos. Pastaba: `_headers` `X-Frame-Options: DENY` prieštarauja dokumentuotam Shopify-iframe embed tikslui — paliktas `DENY` (saugiau); keisti į `SAMEORIGIN`/`frame-ancestors` tik jei embed realiai prireiks.
- **2026-07-05 (3)** — **2 ETAPAS: 4 naujos produktų kategorijos (batch 1).** Iš „OPTISOFA ASORTIMENTAS" aplanko sukelti produktai, kurių dar nebuvo. Pridėtos kategorijos **po** „Modulinės sofos" ir „Foteliai": **Pufai (4), Lovos (4), Sofos-lovos (11), Kampinės sofos (4)** = **23 produktai**.
  - **Konvertavimas:** OBJ→GLB per Python `trimesh` (šioje aplinkoje NĖRA assimp/blender/gltfpack, tik OBJ). Mastelio auto-detekcija (jei bbox>5 → cm→m ×0.01). Sofos-lovoms imtas esamas `.obj` (dažnai „unfolded"=DL/lovos būsena). Failai plokščioje repo šaknyje: `pufas-*`, `lova-*`, `sofa-lova-*`, `kampine-*.glb`.
  - **Kainos:** tikros EUR iš `optisofa.com/en-de/products.json` (min = „nuo" kaina). Tilda pufas + Harris RXL — `price_approx:true` (Optisofa kataloge nerasti). Matmenys iš 3D bbox.
  - **Kodas:** apibendrinta single-product sistema — `PRODUCT_CATS`, `PRODUCTS{}`, `buildProductList`, `loadProduct` (buvęs `loadArmchair` → alias), `catOf/findProduct`; `buildArmSpec(a,specSel)`, `openCat` slepia visų kategorijų spec; share-link/`?p=` deep-link veikia visoms kategorijoms. Nauji `.cat-hd`/`.cat-body` + i18n `catPufai/catLovos/catSlovos/catKampines` (LT/EN/FR). 4 nauji JSON (`pufai/lovos/sofos-lovos/kampines.json`) — ta pati schema kaip `armchairs.json`.
  - **Verifikuota naršyklėje:** visos 6 kategorijos teisinga tvarka; Esme kampinė (23MB) ir Blow pufas (€280) įkelia su audiniu + spec; kainos teisingos; konsolė be klaidų.
  - **LIEKA (kiti batch'ai / tobulinimas):** (a) **FBX-only produktai** be `.obj` (Esme/Joleen/Juliett/Juliett2/Harris sofabed, Jane corner ~7) — reikia assimp/blender (ne šioje aplinkoje); (b) **25 dar `.zip` šeimos** (Abbe, Amy, Clara, Mood, Sunday ir kt., kai kurios 100–270MB) + 1 nebaigtas download; (c) **sunkūs GLB** (`kampine-esme` 23MB, `sofa-lova-jane` 15MB, `sofa-lova-bonnie` 11MB, keli 8–9MB) — reikia gltfpack optimizacijos; (d) **kojų spalvos/aprašymai/features** naujiems produktams — generiniai, tikslintini iš Optisofa; (e) **Sofos** (paprastos) + **Su šezlongu** kategorijos — dar nesukurtos (naudotojas prašė 4). Bendras naujų GLB dydis ~132MB.
- **2026-07-05 (4)** — **Sofos-lovų miego funkcija: „Sofa ↔ Lova" perjungiklis.** Patikslinta: **DL = Dolphin** (Optisofa ištraukiamas miego mechanizmas); atidaryta būsena failuose žymima `_Open` arba „unfolded"; `_M`/`_D` = kairė/dešinė šezlongo pusė. **Sprendimas (patvirtintas su naudotoju):** gyvas dviejų būsenų perjungiklis, numatytoji — sofa (sudėta), kaip Optisofa `3d_openable`.
  - **Kodas:** `loadProduct(a,showAlt)` krauna `glb` (numatytoji) arba `glb_alt`; `buildArmSpec` rodo „Miego funkcija: Dolphin (DL)" + mygtuką „🛏 Atidaryti į lovą / 🛋 Sudėti į sofą" (jei yra `glb_alt`), arba žymę „Rodoma atidaryta lova · sofos vaizdas ruošiamas" (jei `default_state:'lova'` be antros būsenos). Nauji `sofos-lovos.json` laukai: `sleep_mech`, `default_state`, `glb_alt`. i18n LT/EN/FR. CSS `.as-statebtn`.
  - **Realybė:** `trimesh` FBX/3DS **neįkelia** (patvirtinta) → sudėtos (sofos) būsenos, esančios tik FBX, šioje aplinkoje nepagaminamos. Iš 11 sofų-lovų **tik Bonnie turi abi būsenas kaip .obj** → jam perjungiklis veikia GYVAI (sukonvertuota `sofa-lova-bonnie-open.glb`). 4 rodo sofą (Bonnie/Emilly/Nora/Slay), 7 rodo lovą (sudėta būsena = tik FBX) — pažymėti žyme.
  - **Verifikuota naršyklėje:** Bonnie perjungiklis sofa→lova veikia (mygtukas keičiasi, atidaryta lova renderinasi); Blues rodo lova-only žymę + Dolphin info; konsolė be klaidų.
  - **Naudotojui perduota (`KONVERTUOTI-FBX.md`):** 7 sudėtos būsenos FBX → GLB per Blender/assimp (Blues, Blues-chaise, Bonnie-chaise, Esme, Jane, Jane-chaise, Salma). Atsiuntus — įrašau kaip pagrindinę būseną, dabartinį lovos failą perkeliu į `*-open.glb`, perjungiklis įsijungia visiems.
- **2026-07-05 (5)** — **Kojos (ne audinio spalvos) — išspręsta be FBX + FBX statusas.** Naudotojas: naujų produktų kojos buvo audinio spalvos.
  - **Priežastis:** OBJ failai — vienas sulietas mesh'as generiniais grupių pavadinimais (Box/Cylinder), be medžiagų → `classifyLegs` (per mesh) nepagaudavo kojų → visas korpusas (su kojomis) gaudavo audinį. (Foteliai veikė, nes buvo iš FBX su atskirais mesh'ais.)
  - **Sprendimas (be FBX):** naujas konverteris `convert_legs.py` — **rankiniu būdu parsina OBJ `g` grupes** ir skaido į du mesh'us: **„Body"** (audinys) + **„Legs"** (žemos+plonos grupės / `Cylinder` / `leg|foot`). „Legs" pavadinimą pagauna esamas `classifyLegs` (regex `/leg/i`) → `noFab` → `applyLegColor` uždeda spalvą. **Visi 23 perkonvertuoti.** 19 turi atskirtas kojas; 4 be kojų (pufas-slay/tilda, sofa-lova-slay, kampine-slay — heuristika kojų nerado, tikslintina).
  - **Kojų spalvos:** iš verifikuotų fotelių pagal šeimą (Blues/Salma/Uma=juodas metalas; Emilly/Esme/Harris/Juliett=ąžuolas). **`leg_approx:true`** pažymėti tie, kur spalva numatyta (Bonnie/Jane/Nora/Slay-lovos/Blow) — reikia patikrinti Optisofa puslapiuose.
  - **Verifikuota naršyklėje:** Esme sofa-lova — ąžuolo kojos; Blow pufas — juodos kojos; **nebe audinio spalvos.**
  - **FBX apribojimas patvirtintas 3 būdais:** `trimesh` (ne FBX/3DS), `bpy` (nėra Python 3.9), `aspose-3d` (procesą užmuša OOM). Sudėtos (sofos) būsenos, esančios tik FBX, ir toliau nekonvertuojamos čia → 7 sofos-lovos vis dar rodo lovą (numatytoji). **Sprendimas #1 (visos sofos-lovos suskleistos):** 4 turi sudėtą obj → rodo sofą; 7 reikia FBX konvertavimo per Blender (žr. `KONVERTUOTI-FBX.md`). Perjungiklis „Sofa↔Lova" jau paruoštas — įsijungs atsiuntus sudėtas GLB.

- **2026-07-05 (6)** — **FBX ATRAKINTAS + VISAS ASORTIMENTAS SUKELTAS.** Naudotojas patikslino, kad konvertavimą darau aš. Po agresyvaus bandymo rastas veikiantis kelias: **`assimp_py`** (pip wheel su įdėta assimp biblioteka) **skaito FBX ir 3DS** (`bpy` nėra Python 3.9, `aspose-3d` OOM). Pastatytas FBX/3DS→GLB konverteris (`fbx_convert.py` + `master_convert.py`) su node transformacijomis, auto-masteliu (cm→m) ir kojų atskyrimu (Body/Legs).
  - **Sofos-lovos užbaigtos:** sukonvertuotos visos 7 trūkstamos **sudėtos (sofos) būsenos** iš FBX/3DS → **visos 11 sofų-lovų dabar rodo sofą** (numatytoji), 8 su gyvu „Sofa↔Lova" perjungikliu.
  - **Visas Optisofa asortimentas:** 44 šeimos, **163 variantai** → sukonvertuota **~127** (99 nauji + 28 buvę). **176 GLB, 788 MB.**
  - **2 naujos kategorijos UI:** **Sofos (40)** ir **Sofos su šezlongu (11)** — po Kampinių. Iš viso **8 kategorijos, 133 produktai** (Foteliai 27, Pufai 15, Lovos 12, Sofos-lovos 18, Kampinės 10, Sofos 40, Su šezlongu 11).
  - **Kainos:** iš `optisofa.com/products.json` (min). Atitikta 95/133; **38 be kainos (€0, `price_approx`)** — matcher'is per griežtas kai kurioms šeimoms, tikslintina.
  - **HD (užduotis 3):** pixelRatio 2→2.5, šešėliai 2048→4096, audinių tekstūros 1536→2048, miniatiūros 360×270→480×360.
  - **Verifikuota naršyklėje:** visos 8 kategorijos, 133 produktai užsikrauna; Abbe sofa (nauja) renderinasi su kojomis, HD; konsolė be klaidų. Bonnie/Nora foteliai (FBX davė 27cm) pertaisyti iš OBJ.
  - **LIEKA iki „100%":** (a) **38 kainos** + pilna spec verifikacija su Optisofa (pavadinimai/matmenys/kojų apdaila); (b) **sunkūs GLB** (11 vnt >12MB, iš viso 788MB) — reikia `gltfpack` optimizacijos deploy'ui; (c) `verified:false` visiems naujiems — aprašymai/features generiniai; (d) kelios sofos-lovos be atidarytos būsenos (Emilly/Nora/Slay).

- **2026-07-05 (7)** — **GLB optimizacija + kainos.** (a) **Optimizacija:** `gltfpack` neįsidiegė, bet **`pyfqmr`** (quadric decimation, pip) + geometrijos pervalymas → **788MB → 234MB (3.4×)**. Decimuoti tik single-product GLB (foteliai/pufai/lovos/sofos-lovos/kampinės/sofos/sezlongai — jie naudoja boxUV, UV nesvarbus); **moduliniai (HUG/CLOUD/SLAY/LOFT) NELIESTI** (jie naudoja tikrą UV). Body mesh'ai iki ~14–20% trikampių, „Legs" mažos paliktos; didelės mis-klasifikuotos „Legs" (pvz. Elvi lovos pagrindas 319k) irgi decimuotos. Kokybė furniture'ui gera (patikrinta naršyklėje). `DracoPy` yra (tolimesnei kompresijai su DRACOLoader, jei prireiks → ~80MB). (b) **Kainos:** sulieta en-de + main Optisofa `products.json` (431 unik.). **103/133 (77%) su kaina.** Likę **30 be kainos — 8 šeimos (Abbe, Ahne, Amy, Elvi, Flow, Lottie, Tilda, Yoko) + Amelie/Clara foteliai NĖRA Optisofa viešame kataloge** (nauji/nebgaminami) → viešos kainos nėra, reikia naudotojo. `-2v` sofoms priskirta 3v sofos kaina (apytiksliai). (c) **LIEKA iki „100%":** 30 kainų (ne kataloge); pilna per-produkt spec (aprašymai/tikslios kojų apdailos) — `verified:false`.

- **2026-07-05 (8)** — **Draco kompresija — 788MB → 124MB (6.3×), be kokybės praradimo.** `gltfpack` nėra, bet **`DracoPy`** (pip, su įdėta biblioteka) koduoja Draco. Rankinis glTF+GLB kūrėjas (`draco_glb.py`): POSITION+NORMAL, 14-bit kvantizacija (vizualiai lossless), UV nekoduojamas (konfigūratorius generuoja boxUV). Į konfigūratorių pridėtas **DRACOLoader** (dekoderis iš jsdelivr; CSP jau leidžia). **Svarbios pamokos:** (a) GLB JSON dalis pildoma **tarpais 0x20** (ne nuliais); (b) **DracoPy priskiria unique_id: NORMAL=0, POSITION=1** (atvirkščiai nei tikėtasi) — glTF extension `attributes:{POSITION:1,NORMAL:0}`, kitaip normalės skaitomos kaip pozicijos → geometrija „rutulys". Decimuoti (pyfqmr) + Draco taikyta tik single-product GLB; **moduliniai nepaliesti** (tikras UV). **Verifikuota naršyklėje:** kampine-slay/esme, pufas-blow, sofa-amelie dekoduojasi teisingais matmenimis, audinys+kojos+apšvietimas OK. Bendras GLB **124MB** (single-product Draco ~91MB + moduliniai ~33MB).

- **2026-07-05 (9)** — **GYVA SVETAINĖ KABO ties „Kraunama…" — IŠTAISYTA.** Įkėlus Draco versiją, konfigūratorius neužsikraudavo. Diagnozė per naršyklės tinklą/konsolę (gyva svetainė): modulis veikė, bet `init()` kabo. **Priežastis: `_headers` CSP `script-src` neturėjo `'wasm-unsafe-eval'` → Netlify blokavo Draco WASM dekoderį** (lokaliai su python serveriu CSP netaikomas, todėl testuose veikė). `restoreConfig` iš `localStorage` bandydavo įkelti Draco produktą → dekoderis kabo → loaderis niekada nepasislepia. **Du taisymai:** (1) `_headers` `script-src` pridėta **`'wasm-unsafe-eval'`**; (2) `init()` atsparumas — `$('#loader').style.display='none'` perkeltas IŠ KARTO po UI sukūrimo (prieš `restoreConfig`), atstatymas apgaubtas `try/catch` → net jei modelis nepavyksta/kabo, konfigūratorius naudojamas. **Verifikuota lokaliai su CSP taikymu:** loaderis pasislepia, Draco sofa-amelie renderinasi (dygsniuotas atlošas, kojos, teisinga geometrija), konsolė be klaidų. **Naudotojui: reikia perkelti tik 2 failus — `index.html` ir `_headers`.**
- **2026-07-05 (10)** — **QA pataisos po naudotojo peržiūros:** (1) **Foteliai** — visi 27 (+Harris RXL=28) yra meniu su miniatiūromis (patikrinta gyvai; „trūkumas" buvo sena versija). (2) **Lovos** — pašalinti dublikatai (lova-slay/-1/-2, lova-uma = lova-slay-140/160/180, lova-uma-daybed); **Elvi lovų atvaizdavimo klaida IŠTAISYTA** — 70% viršūnių klaidingai pažymėta „Legs" (juoda) → pridėta riba: jei kojos >30%, viskas = Body (audinys); Elvi dabar apmuštas audiniu. (3) **Audinys/apšvietimas** — sumažinta šviesa tikslesnei spalvai: exposure 0.95→0.82, key light 1.35→1.05, hemisphere 0.40→0.30, fill 0.28→0.20, fabric envMapIntensity 0.30→0.20. (4+7) **Sofos-lovos/chaise** — visos → **default sofa (suskleista)**; per pilną konvejerį (convert→decimate→draco) sugeneruotos abi būsenos. (5) **Perjungiklis** „Sofa↔Lova" — 14/18 sofa-lova + 3 sezlongas su gyvu mygtuku (kur yra atidarytos būsenos šaltinis). (6) **Harris RXL** perkeltas iš Kampinių į **Foteliai** (fotelis-harris-rxl). `master_convert.py` apsaugotas `__main__` guard (nebekartoja importuojant). Verifikuota naršyklėje.
- **2026-07-05 (11)** — **Tuščios miniatiūrų pozicijos (foteliai ir kt.) — IŠTAISYTA.** Priežastis: paleidžiant konfigūratorių, VISŲ ~180 produktų miniatiūros generuodavosi vienu metu (kiekviena = Draco dekodas + offscreen render) → eilė persipildydavo, dalis kortelių likdavo tuščios kelias sekundes/neapsigeneruodavo. **Sprendimas: LAZY miniatiūros** — `buildProductList` tik sukuria korteles (su ⏳), o `genThumbs(listId)` generuoja miniatiūras tik **atidarius tą kategoriją** (`openCat`). Taip paleidžiant generuojasi tik matoma modulinė kategorija, o foteliai/sofos/etc. — kai atidaromi. **Verifikuota:** atidarius Foteliai, visi 28 (įsk. harris-rxl) sugeneruoja miniatiūras per ~10s, 0 tuščių. Failas: tik `index.html`.
- **2026-07-05 (12)** — **Foteliai (ir kt.) neužsikraudavo — ROOT CAUSE ir taisymas.** Naudotojas: daug fotelių tuščių (Emilly/Esme/Harris/Ines/Joleen), nauji (Jane/Dope) veikia. Diagnozė: veikiantys = nauji (Body/Legs, mano konvejeris), tušti = **originalūs 11 fotelių iš 1-os sesijos** (dideli, ne-Draco, multi-mesh su 3ds-max node transformacijomis). **Priežastis: `draco_glb.py` iteruodavo `scene.geometry` NEtaikydamas node transformacijų** → multi-mesh su transform'ais sukrisdavo į origin/dingdavo (naujieji transform'ų neturi → veikė). **Taisymas:** `draco_glb` dabar eina per `scene.graph.nodes_geometry` ir taiko kiekvieno node `matrixWorld` prieš koduojant. Originalūs 11 atkurti iš git + per-Draco su transformacijomis (+decimavimas). **Papildomai:** perkonvertuoti VISI likę ne-Draco/dideli single-product GLB (65) į vienodą Body/Legs+Draco. **Verifikuota naršyklėje:** visi **129 single-product produktai užsikrauna, 0 sugadintų** (bbox 0.4–3.5m); Esme/Blues/Harris renderinasi su kojomis. Bendras GLB dydis liko ~123MB.
- **2026-07-05 (13)** — **Asortimento suvienodinimas su Optisofa shopu — FOTELIAI.** Naudotojas: palikti tik tuos, kurie yra Optisofa. Paimta Optisofa fotelių kolekcija (`/en-de/collections/armchairs`) — **17 fotelių**. Mūsų 28 → **16** (visi 16 yra Optisofa). **Ištrinta 12:** 10 ne-Optisofa šeimų (Abbe, Ahne, Amelie, Amy, Clara, Flow, Lottie, Tilda, Tilda 2, Yoko) + Harris RXL (nėra fotelių kolekcijoje) + fotelis-juliett-2 (dublikatas su fotelis-juliett-ii = „Juliett II"). **Trūksta: „Slay armchair"** (Optisofa turi, mes ne — anksčiau nepavyko konvertuoti; galima pridėti). JSON + 12 GLB ištrinti iš repo ir GITHUB-UPLOAD.
- **2026-07-05 (14)** — **Asortimentas suvienodintas su Optisofa shopu (visos kategorijos).** Paimtos Optisofa kolekcijos: pufai←`pouffes`, lovos←`beds`, sofos-lovos←`3dl` (Sleeping function), kampinės←`corner-sofas`, sofos←`sofas`, su šezlongu←`with-chaiselounge`. Match pagal šeimos vardą žodžio riba bet kur pavadinime (nes Optisofa kartais rašo tipą pirmą, pvz. „Stool Slay"). **Ištrinta 22:** Pufai −5 (flow,lottie,tilda,tilda-2,yoko)→10; Lovos −5 (elvi×4,uma-daybed)→3; Sofos-lovos −2 (abbe-chaise,amy)→16; Kampinės −2 (flow,yoko)→7; Sofos −8 (abbe,amelie,amy,yoko ×2v)→32; Su šezlongu −0→11. + Foteliai (13 įraše) →16. **Galutinis: Foteliai 16, Pufai 10, Lovos 3, Sofos-lovos 16, Kampinės 7, Sofos 32, Su šezlongu 11.** GLB 123→92MB. JSON + 35 GLB (12+23) ištrinti iš repo ir GITHUB-UPLOAD. **Trūkstami (Optisofa turi, mes ne — reikia 3D failų):** Slay armchair (tuščias failas); Lovos: Molly, Spark; Pufai: Bowie, Loft, Taku; ir kt.
- **2026-07-05 (15)** — **Lovos: pridėti Molly+Spark, ištaisytas vaizdas.** Naudotojo failai. (1) **Molly (4: 90/140/160/180) + Spark (2: 160/180)** sukonvertuoti (OBJ, wood variantas Molly), kainos iš `beds` kolekcijos → **Lovos = 9 (= Optisofa: Slay 3, Molly 4, Spark 2).** (2) **Slay lovos vaizdas** buvo neteisingas — rodė atvirą sandėliavimo vidų (Bed_Inner, Angle_Bar, slats). Perkonvertuota **be vidaus mechanizmo** (palikta išorė+Mat+Glides). (3) **Procedūrinis čiužinys** visoms lovoms (`addMattress` per raycast randa slat lygį + uždeda čiužinį) — pagrindiniame vaizde IR miniatiūrose (`renderThumb`). Dabar lovos atrodo kaip Optisofa (su čiužiniu, ne tuščias rėmas). Verifikuota: Slay/Molly renderinasi su čiužiniu, konsolė švari.
- **2026-07-06 (4)** — **Lovų pataisos po naudotojo pastabų.** (a) Miniatiūros nebeiškraipytos: `renderThumb` nebeprideda procedūrinio čiužinio, jei modelis turi savą (mesh „Mattress"). (b) **„frame" pašalintas iš bendro kojų regex** — Molly/Spark cokolis („Frame" mesh) yra APMUŠTAS, todėl gaudavo kojų spalvą; dabar cokolis = audinys, kaip galvūgalis. Dope rėmas juodas paliktas per per-produkto `legNames:"frame"`. (c) Spark kojos → ąžuolas (JSON). Verifikuota: Molly/Spark cokolis = audinys + ąžuolo kojos; Dope rėmas juodas; lovų miniatiūros teisingos.
- **2026-07-06 (3)** — **Probleminių modelių PERKONVERTAVIMAS iš šaltinio (OPTISOFA ASORTIMENTAS).** Esminis atradimas: „sulieta Body" problema kilo iš MANO konverterio (`fbx_convert.py` suliedavo viską į vieną mesh'ą). Naujas `reconv.py` IŠSAUGO atskirus mesh'us (assimp objektų vardai) + gali IŠMESTI vidines dalis. Pipeline: assimp → trimesh.Scene → decimate (pyfqmr) → plain GLB → `draco_glb.py`. Verifikuota naršyklėje (`_view.html`) + realiame konfigūratoriuje:
  - **Harris kampinė** → naudotas SUSKLEISTAS šaltinis („Narożnik…z funkcją spania, lewy" – ne „rozłożony"); dabar teisinga L formos suskleista sofa + ąžuolo kojos. **Joleen šezlongas** → suskleistas šaltinis.
  - **Clara** (sofa 3v/2v/sofa-lova) → atlošinės pagalvės nebeperšviečia (senas konvertas jas sugadindavo); pilnavertė sofa su dekoratyvinėm pagalvėm + juodos kojos.
  - **Blow** (fotelis/sofa/lounge) → atskiras „legs" mesh → ąžuolo kojos. **Dope** → atskiras „Frame" mesh → juodas metalinis rėmas.
  - **Visos lovos** (Slay 140/160/180, Molly 90/140/160/180, Spark 160/180) → švarus apmuštas pagrindas + galvūgalis + modelio savas čiužinys (mesh „Mattress" nudažomas šviesiai, ne audiniu; procedūrinio nebepridedam) + kojos. Neberodo grotelių/tarpo. Vidinės dalys (Wood/Construction/Screws/Bed_Inner/Holder…) išmestos per `keep` baltąjį sąrašą.
  - **index.html:** leg-name regex išplėstas (`frame|glid|crossbar|black_metal|koj`); `addMattress` aptinka modelio savą čiužinį (mesh „Mattress") ir jį nudažo šviesiai; **GLB_V cache-bust** (`?v=20260706b`) — atnaujinus GLB naršyklė paima naują. Mia: per-produkto `legNames` (`^(Line|Cylinder)`).
- **2026-07-06 (2)** — **22-punktų pataisymų sąrašas (kojos, dublikatai, kategorijos).**
  - **JSON duomenys:** ištrinti dublikatai — Juliett 2 pufas, Slay pufas (dublis), Juliett 2 sofa-lova, Slay kampinė (turime prie modulinių), Mood „tik šezlongas" modulis (`sezlongas-mood` W95). Harris RXL perkeltas iš kampinių į **Foteliai** (`fotelis-harris-rxl`). Blow fotelio/sofos kojos JSON → ąžuolas.
  - **Kojų rendering (esminis):** modeliai su **sulieta viena „Body" mesh'a** (Karin, Uma, Salma, Jane, Blues...) neturėjo atskiro kojų mesh'o → kojos gaudavo audinio spalvą. Sprendimas: `computeLegY()` aptinka **tuščią tarpą** tarp kojų galų (apačia) ir apmušalo, o fabric shader'is (`applyFabricTo`) nudažo zoną žemiau `uLegY` kojų spalva (metalas/ąžuolas) per `vWP.y` — **be geometrijos skaldymo**. Verifikuota naršyklėje: Karin, Uma → juodos metalinės kojos; Emilly → ąžuolas. Mia metalinis rėmas (Line/Cylinder mesh'ai) — naujas per-produkto `legNames` regex → juodas. Nora jau turėjo atskirą „Legs" mesh'ą (veikė).
  - **NEIŠSPRĘSTA (reikia 3D modelių perkonvertavimo iš šaltinio):** visų lovų iškraipymas (Slay bazė rodo groteles + tarpą), Clara sofa-lova/2v atlošinės pagalvės permatomos (Body mesh geometrija), Blow kojų ąžuolas (suliета mesh be tarpo — negalima atskirti), Dope metalinis rėmas (sulietas), Harris kampinė ir Joleen šezlongas rodomi **išskleisti** (nėra suskleisto modelio). Žr. skyrių „Atviri sprendimai".
- **2026-07-06 (1)** — **Audinys SUPER HD + realistiškas rendering.** Naudotojas: audinys turi būti kaip super HD (matoma tekstūra priartinus), be netolygumo, realu kaip tikra sofa; apšvietimas tikroviškas. **Esminis pakeitimas: `boxUV` (kietas per-viršūnę projekcija → siūlės ant apvalių paviršių) pakeistas TIKRU TRIPLANAR shader'iu** (`applyFabricTo` `onBeforeCompile`): 3 world-space projekcijos sklandžiai maišomos pagal normalę (`pow(abs(n),4)`), audinys pertoninamas iš sRGB, weave gylis per roughness moduliaciją. **Rezultatas: jokių siūlių/netolygumo, matoma reali weave/boucle tekstūra priartinus.** Tekstūros: **MirroredRepeatWrapping** (siūlės nesimato), max anisotropy, mipmaps, 1600px, tile 0.34m (realus weave mastelis). `fabricParams` sheen/roughness pagal kategoriją (velvet=aksomo sheen, boucle=faktūra). Apšvietimas: fabric `envMapIntensity` 0.20→0.55 (soft IBL), exposure 0.82, PMREM RoomEnvironment, PCFSoft 4096. **Audinių kolekcija:** pridėti 6 nauji Optisofa audiniai (Aqua, Blush, Brick, Desert, Natural, Teddy) → 81. **Verifikuota:** Blues boucle rodo kilpas, Forest Velvet — aksomo blizgesį; be siūlių, fotorealistiška.