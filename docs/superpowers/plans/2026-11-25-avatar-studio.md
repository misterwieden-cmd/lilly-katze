# Avatar-Studio Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Lilly (3D-Katze), Fritz (3D-Gehirn) und einen menschlichen Avatar als zuschaltbare 3D-Figuren ins bestehende Lilly-Dashboard bringen, plus eine Studio-Seite zum Importieren und Prüfen von GLB/VRM-Modellen – ohne bestehende Funktionen zu ersetzen.

**Architecture:** Der SVG-Katzenkopf bleibt Standard. Ein neuer Knopf `FIGUR` lädt per `import()` erst bei Bedarf ein three.js-Modul, das sich an die vorhandenen Stellen `setZustand()` und `pegelLesen()` hängt. Reine Logik (Zustand→Clip, Mund, Budget, fps-Wächter) liegt getrennt und ist in Node testbar. Bezahlte 3D-Generierung passiert nie im Browser, sondern später im PC-Backend hinter einer Freigabe.

**Tech Stack:** three.js 0.170.0 (lokal unter `vendor/three/`), GLTFLoader + meshopt-Decoder, später `@pixiv/three-vrm`; Tests mit `node --test` und der vorinstallierten Playwright-Bibliothek; Asset-Nachbearbeitung in Blender (kostenlos).

**Spec:** `docs/avatar-studio/analyse.md`

## Global Constraints

- Nichts ersetzen: `localStorage.lilly_figur` ist standardmäßig `svg`; WebSocket-, HTTP-, Audio- und Sprachlogik in `index.html` bleiben unverändert.
- three.js exakt `0.170.0`, lokal unter `vendor/three/`, nur per dynamischem `import()` geladen. Zur Laufzeit kein CDN.
- Keine API-Keys in Dateien, die der Browser lädt.
- Kostenregel: kein bezahlter API-Aufruf ohne vorheriges Angebot (Betrag + Zweck) und ausdrückliche Freigabe von Mister. Gratis-Läufe werden trotzdem protokolliert.
- `BUDGET_INTEL = { dreiecke: 30000, knochen: 64, texturMaxPx: 1024, dateiBytes: 3000000 }`; Fritz zusätzlich ≤ 10.000 Dreiecke.
- Renderer: `alpha: true` (Strahlenkranz `#strahlen` bleibt sichtbar dahinter), Pixelratio `Math.min(devicePixelRatio, 1.5)`, keine Schatten, kein Post-Processing, max. 30 fps, keine Frames bei `document.hidden`, `stop()` ruft `renderer.dispose()` und `renderer.forceContextLoss()`.
- Zustandsfarben aus `index.html:392-393`: idle/listening `67,232,255`, speaking `255,95,162`, thinking `139,92,246`, error `255,84,112`.
- UI-Texte deutsch, Knopfbeschriftung im bestehenden Stil: `FIGUR SVG`, `FIGUR FRITZ`, `FIGUR LILLY`, `FIGUR MENSCH`.
- Fallback-Meldungen als Meta-Blase mit Präfix `3D aus: ` (z. B. `3D aus: kein-webgl2`).

## Review Focus

1. Kein WebGL2 (alter Intel-Treiber, GPU-Sperrliste) → SVG-Kopf bleibt, Meta-Blase `3D aus: kein-webgl2`, kein Absturz. *(Task 4, Test C)*
2. GLB fehlt oder ist kaputt → zurück zu SVG, Meta-Blase `3D aus: laden-fehlgeschlagen…`. *(Task 4, Test E)*
3. Intel-Grafik zu langsam (< 24 fps über 3 s) → automatisch zurück zu SVG, `3D aus: zu langsam`. *(Task 1 Wächter-Test, Task 4 Verkabelung)*
4. Figur 20× hin- und herschalten → nie mehr als ein 3D-Canvas, keine Warnung „Too many active WebGL contexts“. *(Task 4, Test D)*
5. Unbekannter oder schnell wechselnder Zustand → wird wie `idle` behandelt, Clip-Überblendung ohne Fehler. *(Task 1, `normZustand`-Test)*

---

## Phase 0 – Vorab-Checks (Mister/AutoClaw, 0 €)

- [ ] Auf dem Intel-Laptop `chrome://gpu` öffnen: steht bei WebGL2 „Hardware accelerated“? GPU-Name notieren und in `docs/avatar-studio/analyse.md` §1 eintragen.
- [ ] Auf dem PC prüfen, ob HUD/Basis schon three.js nutzt:
  `Get-ChildItem C:\Users\Christan\Documents\Lillys-Imperium -Recurse -Include *.html,*.js -ErrorAction SilentlyContinue | Select-String -Pattern "from 'three'|THREE\.|three\.module" -List | Select-Object Path`
  Treffer → dortige Version notieren; weicht sie ab, entscheidet Mister, welche Version beide nutzen.

## Phase 1 – 3D-Bühne, Fritz und Studio (Code, 0 €)

### Task 1: Logik-Modul

**Files:**
- Create: `package.json` (nur Dev-Tools: `"type": "module"`, Script `"test": "node --test tests/"`)
- Create: `avatar/logik.js`
- Test: `tests/logik.test.mjs`

**Interfaces:**
- Produces:
  - `ZUSTAENDE = ['idle','listening','thinking','speaking','error']`
  - `BUDGET_INTEL` (Wert siehe Global Constraints)
  - `normZustand(z: any) -> string` – unbekannt/leer → `'idle'`
  - `clipFuerZustand(eintrag: {clips?: object}, zustand: string) -> string|null` – `clips[zustand]`, sonst `clips.idle`, sonst `null`
  - `mundOeffnung(pegel: number, zustand: string) -> number` – Pegel auf [0,1] begrenzen; speaking: `0.15 + 0.85·p`; error: `0`; sonst `0.3·p`
  - `pruefeBudget(stats: object, budget: object) -> {ok: boolean, verstoesse: string[]}` – Meldung `"<schluessel> <ist> > <max>"`, fehlende Stat-Werte ignorieren
  - `erstelleFpsWaechter({minFps = 24, dauerMs = 3000}) -> {tick(jetztMs: number): 'ok'|'zu_langsam'}`
  - `naechsteFigur(aktuell: string, verfuegbar: string[]) -> string` – Reihenfolge `['svg', ...verfuegbar]`, zyklisch; unbekannte aktuelle Figur → `'svg'`

- [ ] **Step 1: Failing tests schreiben** (`tests/logik.test.mjs`, `node:test` + `node:assert/strict`)

```js
assert.equal(normZustand('speaking'), 'speaking');
assert.equal(normZustand('quatsch'), 'idle');
assert.equal(normZustand(undefined), 'idle');
assert.equal(clipFuerZustand({clips:{idle:'Idle', speaking:'Sprechen'}}, 'speaking'), 'Sprechen');
assert.equal(clipFuerZustand({clips:{idle:'Idle'}}, 'thinking'), 'Idle');
assert.equal(clipFuerZustand({}, 'idle'), null);
assert.equal(mundOeffnung(0, 'speaking'), 0.15);
assert.equal(mundOeffnung(2, 'speaking'), 1);
assert.equal(mundOeffnung(-1, 'speaking'), 0.15);
assert.equal(mundOeffnung(1, 'idle'), 0.3);
assert.equal(mundOeffnung(0.5, 'error'), 0);
assert.deepEqual(pruefeBudget({dreiecke:31000, knochen:10, texturMaxPx:1024, dateiBytes:100}, BUDGET_INTEL),
  {ok:false, verstoesse:['dreiecke 31000 > 30000']});
assert.deepEqual(pruefeBudget({dreiecke:12}, BUDGET_INTEL), {ok:true, verstoesse:[]});
// fps-Wächter: Bewertung erst nach 1000 ms Daten; zu_langsam, sobald fps (Frames der letzten 1000 ms)
// seit >= dauerMs ununterbrochen < minFps
const lahm = erstelleFpsWaechter({}); let r;
for (let t = 0; t <= 3000; t += 50) r = lahm.tick(t);   assert.equal(r, 'ok');
for (let t = 3050; t <= 4100; t += 50) r = lahm.tick(t); assert.equal(r, 'zu_langsam');
const flott = erstelleFpsWaechter({}); let s = 'ok';
for (let t = 0; t <= 5000; t += 16) if (flott.tick(t) !== 'ok') s = 'zu_langsam'; assert.equal(s, 'ok');
const aus = erstelleFpsWaechter({minFps:0});
for (let t = 0; t <= 8000; t += 500) r = aus.tick(t);   assert.equal(r, 'ok');
assert.equal(naechsteFigur('svg', ['fritz']), 'fritz');
assert.equal(naechsteFigur('fritz', ['fritz']), 'svg');
assert.equal(naechsteFigur('lilly', ['fritz']), 'svg');
```

- [ ] **Step 2:** `npm test` → FAIL (Modul fehlt)
- [ ] **Step 3:** `avatar/logik.js` implementieren (ES-Modul, keine Abhängigkeiten, kein DOM)
- [ ] **Step 4:** `npm test` → alle PASS
- [ ] **Step 5:** Commit `feat(avatar): Logik fuer Zustaende, Mund, Budget, fps-Waechter`

### Task 2: three.js lokal + 3D-Bühne + Fritz

**Files:**
- Modify: `package.json` – devDependency `three@0.170.0`, Script `"vendor": "node tools/vendor-three.mjs"`
- Create: `tools/vendor-three.mjs` – kopiert aus `node_modules/three/`: `build/three.module.min.js` und aus `examples/jsm/`: `loaders/GLTFLoader.js`, `utils/BufferGeometryUtils.js`, `libs/meshopt_decoder.module.js`, `controls/OrbitControls.js` nach `vendor/three/` (Addons unter `vendor/three/addons/` mit gleicher Unterstruktur)
- Create: `avatar/buehne.js`, `avatar/fritz.js`, `avatare/manifest.json`
- Create: `tests/e2e/helfer.cjs` (startet `python3 -m http.server 8090 --bind 127.0.0.1` im Repo-Root, startet Chromium über `require('playwright')`)
- Create: `tests/e2e/buehne.html` (Testseite, siehe Step 1)
- Test: `tests/e2e/buehne.test.cjs`

**Interfaces:**
- Consumes: `normZustand`, `clipFuerZustand`, `mundOeffnung`, `erstelleFpsWaechter` aus Task 1
- Produces:
  - Importmap (in jeder Seite, die die Bühne lädt): `{"imports":{"three":"./vendor/three/three.module.min.js","three/addons/":"./vendor/three/addons/"}}`
  - `erstelleBuehne(container: HTMLElement, opt: {eintrag: object, minFps?: number, onZuLangsam?: () => void, orbit?: boolean}) -> Promise<Buehne>`
    - wirft `Error('kein-webgl2')`, wenn `getContext('webgl2')` fehlt; `Error('laden-fehlgeschlagen: <url>')` bei Ladefehler
    - `eintrag.prozedural === 'fritz'` → Fritz; sonst `eintrag.datei` per GLTFLoader (mit `MeshoptDecoder`)
  - `Buehne = { setZustand(z), setPegel(p), clips(): string[], spieleClip(name), stats(): {fps, dreiecke, drawCalls, knochen, texturMaxPx}, stop() }`
    - GLB: `AnimationMixer`, Überblendung 0,25 s auf `clipFuerZustand(eintrag, normZustand(z))`; Morph `eintrag.morphs.mund` = `mundOeffnung(pegel, zustand)`; Morph `eintrag.morphs.blinzeln` zufällig alle 2,6–5,8 s für 130 ms (wie SVG-Kopf)
  - `baueFritz(THREE) -> {objekt: THREE.Object3D, update(dtSek, zustand, pegel), dreiecke(): number}` – zwei gefurchte Hirnhälften mit Fresnel-Rand-Leuchten in der Zustandsfarbe, Synapsen-Punkte; idle langsames Pulsieren, thinking schnelle Pulse + flackernde Synapsen, speaking Skalierung mit Pegel, error rot und gedimmt
  - `avatare/manifest.json`:
    ```json
    {
      "fritz":  { "aktiv": true,  "prozedural": "fritz" },
      "lilly":  { "aktiv": false, "datei": "avatare/lilly.glb",
                  "clips": { "idle": "Idle", "listening": "Lauschen", "thinking": "Denken", "speaking": "Sprechen", "error": "Fehler" },
                  "gesten": ["Winken"], "morphs": { "mund": "Mund_Auf", "blinzeln": "Blinzeln" } },
      "mensch": { "aktiv": false, "datei": "avatare/mensch.vrm", "format": "vrm" }
    }
    ```

- [ ] **Step 1: Failing e2e-Test** – Testseite `tests/e2e/buehne.html` (Importmap + `erstelleBuehne(div, {eintrag:{prozedural:'fritz'}, minFps:0})`, legt Ergebnis auf `window.__b`). Chromium-Args `['--use-angle=swiftshader','--enable-unsafe-swiftshader']`. Assertions: genau 1 `canvas` im Container; `__b.stats().dreiecke` > 0 und ≤ 10000; nach `__b.stop()` 0 Canvas im Container.
- [ ] **Step 2:** `node --test tests/e2e/` → FAIL
- [ ] **Step 3:** `npm i -D three@0.170.0 && npm run vendor`, dann `buehne.js`, `fritz.js`, `manifest.json` implementieren
- [ ] **Step 4:** `npm test && node --test tests/e2e/` → PASS
- [ ] **Step 5:** Commit `feat(avatar): three.js-Buehne mit prozeduralem Fritz` (inkl. `vendor/three/`)

### Task 3: Studio-Seite

**Files:**
- Create: `studio.html`, `avatar/studio.js`
- Create: `tests/fixtures/baue-test-glb.mjs` (erzeugt mit `@gltf-transform/core` 4.x, exakt pinnen, `tests/fixtures/wuerfel.glb`: Würfel mit 12 Dreiecken und einer Animation `Idle`)
- Test: `tests/e2e/studio.test.cjs`

**Interfaces:**
- Consumes: `erstelleBuehne` (mit `orbit: true`), `pruefeBudget`, `BUDGET_INTEL`
- Produces: Seite mit Datei-Eingabe `#datei` und Drag-and-drop (`.glb`; `.vrm` ab Task 7), Element `#stats` (Zeilen `Dreiecke: <n>`, `Knochen: <n>`, `Textur max: <n> px`, `Datei: <n> KB`, `fps: <n>`), Budget-Ampel `#budget` (`OK` oder die Verstöße aus `pruefeBudget`), Knöpfe für die 5 Zustände, Pegel-Regler `#pegel` (0..1), Knopf je Clip aus `clips()` (`button[data-clip="<name>"]`), Checkbox `#overlay` legt `avatare/referenz/lilly-master.jpg` mit Deckkraft 0,5 über die Bühne, Knopf `#master-ansicht` setzt die Kamera frontal. Dateien bleiben lokal (Object-URL), kein Upload.

- [ ] **Step 1: Failing e2e-Test** – `studio.html` laden, `setInputFiles('#datei', 'tests/fixtures/wuerfel.glb')`; erwarten: `#stats` enthält `Dreiecke: 12`, `#budget` enthält `OK`, `button[data-clip="Idle"]` existiert; `#overlay` anklicken → Overlay-Bild sichtbar.
- [ ] **Step 2:** FAIL bestätigen
- [ ] **Step 3:** Fixture-Skript ausführen, `studio.html` + `avatar/studio.js` implementieren
- [ ] **Step 4:** PASS bestätigen
- [ ] **Step 5:** Commit `feat(avatar): Studio-Seite fuer GLB-Import und Pruefung`

### Task 4: Einbau ins Dashboard

**Files:**
- Modify: `index.html` – `<head>`: Importmap; `#leiste` (`index.html:88-93`): Knopf `<button class="knopf" id="figur" title="Figur wechseln">FIGUR SVG</button>` nach `HUD`; `#buehne` (`index.html:96-110`): `<div id="buehne3d" hidden></div>` (absolut, `inset:0`, über `#strahlen`); `setZustand` (`index.html:144`): zusätzlich `S.b3d.setZustand(z)`, falls aktiv; `zeichnen` (`index.html:388`): nach `pegelLesen()` zusätzlich `S.b3d.setPegel(pegel)`, falls aktiv
- Test: `tests/e2e/figur.test.cjs`

**Interfaces:**
- Consumes: `erstelleBuehne`, `naechsteFigur`, `avatare/manifest.json` (verfügbar = Einträge mit `aktiv: true`)
- Produces: `figurSetzen(name)` im bestehenden IIFE – stoppt alte Bühne; `svg` → `#kopf` sichtbar, `#buehne3d` versteckt; sonst `import('./avatar/buehne.js')` und `erstelleBuehne(...)`, bei Erfolg `#kopf` ausblenden; bei Fehler oder `onZuLangsam` → zurück auf `svg`, Meta-Blase `3D aus: <grund>` (`zu langsam` beim Wächter), `localStorage.lilly_figur = 'svg'`. `minFps` aus `localStorage.lilly_3d_fpsmin` (Standard 24).

- [ ] **Step 1: Failing e2e-Tests**
  - A (Standard): `index.html` laden → `#kopf` sichtbar, `#buehne3d` versteckt, `#figur` zeigt `FIGUR SVG`, keine Konsolenfehler
  - B (SwiftShader, `lilly_3d_fpsmin='0'`): `#figur` klicken → binnen 5 s `#buehne3d canvas` vorhanden, `#kopf` unsichtbar, `FIGUR FRITZ`; erneut klicken → kein Canvas, `#kopf` sichtbar, `FIGUR SVG`
  - C (Args `--disable-webgl --disable-3d-apis`): `#figur` klicken → `#kopf` sichtbar, `FIGUR SVG`, `.blase.meta` enthält `3D aus: kein-webgl2`
  - D: 20× klicken → nie mehr als 1 Canvas, keine Konsolenmeldung mit `Too many active WebGL contexts`
  - E: `page.route('**/avatare/manifest.json')` liefert `lilly` mit `aktiv: true`, `datei: 'avatare/fehlt.glb'`; `lilly_figur='lilly'` vorbelegt → nach Laden `#kopf` sichtbar, Meta-Blase enthält `laden-fehlgeschlagen`
- [ ] **Step 2:** FAIL bestätigen
- [ ] **Step 3:** Änderungen in `index.html` umsetzen; Wiederherstellung von `lilly_figur` beim Start
- [ ] **Step 4:** Alle Tests PASS (`npm test && node --test tests/e2e/`)
- [ ] **Step 5:** Commit `feat(dashboard): zuschaltbare 3D-Figur, SVG bleibt Standard`
- [ ] **Step 6 (Mister, Intel-Laptop):** Dashboard öffnen, `FIGUR FRITZ`, 60 s laufen lassen; fps aus `studio.html` mit Fritz notieren. Unter 30 fps → Budgets in `BUDGET_INTEL` senken, bevor Phase 2 startet.

## Phase 2 – Lilly als 3D-Figur (Kosten-Gates)

### Task 5: Rohmodell erzeugen

- [ ] **5.1 Ansichten-Blatt:** Aus `avatare/referenz/lilly-master.jpg` Seiten-, Rücken- und Dreiviertelansicht erzeugen, alle mit den Merkmalen aus Analyse §2, Prompt-Zusatz „same character as reference image, silver-grey tabby, blue eyes, white paws, dark tail, 3D animated film style“. Bezahltes Werkzeug → erst Angebot + Freigabe. Ablage `avatare/referenz/ansichten/`.
- [ ] **5.2 Gratis-Test Meshy:** Vorher Lizenz der Free-Tier-Ausgaben prüfen und in der Analyse vermerken. Masterbild → GLB, höchstens 5 Versuche (100 Free-Credits, 20 pro Bild→3D). Jedes Ergebnis in `studio.html` mit Overlay prüfen; Notizen in `docs/avatar-studio/testlauf-lilly.md` (Versuch, Merkmale erfüllt ja/nein, Dreiecke).
- [ ] **5.3 GATE – bezahlter Lauf:** Nur, wenn 5.2 die Merkmale nicht trifft. Angebot an Mister: Tripo Multi-View mit den Ansichten aus 5.1 (Preis vorher an der Tripo-Preistabelle bestätigen; Bild→Modell texturiert laut Doku 30 Credits ≈ 0,30 $) oder Rodin (ca. 0,40 $). **Deckel: 5 € gesamt für Phase 2.** Erst nach Freigabe ausführen.

### Task 6: Nachbearbeitung, Rig, Abnahme

- [ ] **6.1 Blender-Skript** `tools/blender/lilly_rig.py`, Aufruf `blender -b avatare/roh/lilly_roh.glb -P tools/blender/lilly_rig.py`: Dezimieren auf ≤ 30.000 Dreiecke; Augen als eigene glänzende Kugeln mit Lidern; Leicht-Rig mit Knochen `Becken, Wirbel_1..3, Hals, Kopf, Kiefer, Ohr_L/R (je 2), Schwanz_1..6, Lid_L/R, Vorderbein_L/R (je 3)`; Shape Keys `Mund_Auf`, `Blinzeln`; Clips `Idle` (Atmen, Schwanzwedeln), `Lauschen` (Ohren vor, Kopf schräg), `Denken` (Blick nach oben, Schwanz kringeln), `Sprechen` (leichtes Nicken; Mund kommt über den Pegel), `Fehler` (Ohren flach), `Winken` (Pfote heben); Export `avatare/lilly.glb` mit meshopt. Gewichte automatisch; sitzen sie schlecht, korrigiert Mister oder ein Fiverr-Auftrag (Preis vorher einholen).
- [ ] **6.2 Abnahme:** `studio.html` → `#budget` zeigt `OK`; Overlay-Vergleich mit Masterbild in `#master-ansicht`; alle 6 Clips abspielbar; ≥ 30 fps über 60 s auf dem Intel-Laptop; **Mister bestätigt: „Das ist Lilly.“**
- [ ] **6.3:** In `avatare/manifest.json` `lilly.aktiv = true`; e2e-Test B um `FIGUR LILLY` erweitern; Commit `feat(avatar): Lilly als 3D-Figur`.
- [ ] **6.4 Optional – GATE:** Stehende Lilly für Shorts: Tripo-Rig-Check (gratis), Quadruped-Rig mit Rigger v2.5 (25 Credits ≈ 0,25 $), Gehen und Laufen (je 10 Credits ≈ 0,10 $). Angebot + Freigabe.

## Phase 3 – Menschlicher Avatar (0 €)

### Task 7: VRM-Avatar

- [ ] **7.1 (Mister):** In VRoid Studio (kostenlos) den Avatar bauen; Export VRM 1.0 mit Polygon-Reduktion und Textur-Atlas 1024 px → `avatare/mensch.vrm`; in `studio.html` prüfen, nachdem 7.2 fertig ist.
- [ ] **7.2 Code:** `@pixiv/three-vrm` in einer zu three 0.170.0 kompatiblen Version exakt pinnen und vendorn; in `erstelleBuehne` Zweig `eintrag.format === 'vrm'`: Ausdruck `aa` = `mundOeffnung(pegel, zustand)`, `blink` zufällig wie in Task 2, Kopfnicken bei `speaking`, Kopfneigung bei `listening`. Studio nimmt `.vrm` an. Test: `tests/e2e/vrm.test.cjs` mit Misters Datei (lokal, nicht in CI, wenn die Datei fehlt → Test überspringen mit Hinweis).
- [ ] **7.3:** `mensch.aktiv = true`; Commit `feat(avatar): menschlicher VRM-Avatar`.
- [ ] **7.4 Optional – GATE:** Fotobasierter Avatar (Avaturn/MetaPerson) nur nach Preis- und Datenschutzprüfung und Freigabe.

## Phase 4 – Studio-Automatisierung im PC-Backend (später, AutoClaw)

Der Backend-Code liegt nicht in diesem Repo; Details legt AutoClaw nach Sichtung von `Lillys-Imperium` fest.

- [ ] Python-Modul `avatar_studio/` mit Anbieter-Adaptern (`tripo`, `meshy`, optional `scenario` mit Dry-Run): `angebot(auftrag) -> {betrag_usd, posten}`, `starte(auftrag, freigabe_id)`, `status(job_id)`, `lade(job_id, ziel)`. Keys nur in `.env` (`TRIPO_API_KEY`, `MESHY_API_KEY`).
- [ ] Freigabe läuft über Lilly (Dashboard/Telegram): Angebot anzeigen, `starte` nur mit gültiger `freigabe_id`. Kein Auto-Approve.
- [ ] Qualitätstor `tools/glb-check.mjs` (gltf-transform, nutzt `pruefeBudget`) für jedes heruntergeladene GLB.
- [ ] Port nicht 8766 (AutoClaw); in `MASTER_STARTEN.bat` einhängen.
