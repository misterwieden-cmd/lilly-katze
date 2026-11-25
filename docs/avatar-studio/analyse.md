# Avatar-Studio – Analyse (Stand 2026-11-25)

Ziel: ein integriertes Avatar-Studio für Lillys Imperium. Priorität: **Lilly** als wiedererkennbare 3D-Katze, **Fritz** als 3D-Gehirn, **menschlicher Avatar** für Mister. Nichts Bestehendes wird ersetzt. Bezahlte Generierungen nur nach Kostenprüfung und Freigabe.

Legende: **belegt** = aus offizieller Doku/Preisseite (laut Suchergebnis), **Drittquelle** = Review/Reseller, **unbekannt** = nicht verifizierbar.

---

## 1. Ausgangslage im Repo

| Punkt | Befund |
|---|---|
| Dashboard-Technik | `index.html` = **SVG-Katzenkopf + Canvas-2D-Strahlenkranz**. Kein three.js, kein WebGL. |
| Andockstellen | `setZustand(z)` (`index.html:144`) mit den Zuständen `idle / listening / thinking / speaking / error`; `pegelLesen()` liefert Audiopegel 0..1 (`index.html:368`); Render-Schleife `zeichnen()` (`index.html:386`). Genau diese drei Stellen reichen, um eine 3D-Figur anzusteuern. |
| HUD / Basis | `/hud` und `/basis` liefert das PC-Backend (Tailscale). Code liegt **nicht** im Repo. Die PC-Ordner (`Lillys-Imperium` u. a. auf LAPI) sind für Cloud-Sessions nicht freigegeben → **ob dort schon three.js läuft, ist unbekannt.** |
| Nutzung | PWA im Hochformat mit Push-to-Talk → läuft auch auf dem Handy. 3D muss abschaltbar sein und darf den Akku nicht leersaugen. |
| Zielgerät | Samsung-Laptop „LAPI“ (Windows) mit Intel-Grafik. Genaues Modell und GPU **unbekannt** – auslesen per PowerShell: `Get-CimInstance Win32_ComputerSystem \| Select-Object Manufacturer, Model` und `Get-CimInstance Win32_VideoController \| Select-Object Name, DriverVersion`. Budgets unten sind Startwerte und müssen auf dem Gerät gemessen werden. |

## 2. Referenz Lilly – Wiedererkennungsmerkmale

Masterbild: `avatare/referenz/lilly-master.jpg` (1024×1024).

- Silbergraues Fell mit schwarzen Tabby-Streifen (Stirnzeichnung, Wangen, Rücken, Beine)
- Große, glänzende **blaue Augen** mit dunklem Lidstrich
- Rosa Nase, rosa Innenohren, große Ohren mit hellen Fellbüscheln
- Weiße Brust-Halskrause, weiße Pfoten
- Buschiger Schwanz mit schwarzen Ringen und schwarzer Spitze
- Proportionen: großer Kopf zu kleinem Körper (Kindchenschema), **sitzende** Pose, 3D-Animationsfilm-Look

Hinweis: Im Imperium-Kontext steht „Ragdoll-Katze“ – das Masterbild zeigt eine **silbergraue Tabby**. Für das 3D-Modell gilt das Masterbild.

## 3. Geprüfte Werkzeuge

### 3.1 Die drei angefragten

**three.ws 3D Skills** – Claude-Code-Skills (`generate-3d-model`, `create-3d-avatar`, `rig-a-model`, `find-3d-assets`, `embed-three-ws-avatar`).
- Text→3D kostenlos (TRELLIS über NVIDIA NIM), Ausgabe GLB. **Text-only** – kein Foto-Upload im Gratis-Lane.
- Foto→Avatar nur über `create-3d-avatar`, und das ist **ausschließlich humanoid**; Vierbeiner werden ausdrücklich abgewiesen. `rig-a-model` setzt ebenfalls eine humanoide Figur voraus.
- Das Foto-Lane läuft laut Skill über Hunyuan3D. Die Tencent-Lizenz für Hunyuan3D schließt laut Drittquelle **EU/UK/Südkorea vom Lizenzgebiet aus** – für Mister in Deutschland ein Problem.
- Benötigt three.ws-Account/API-Key (die README sagt „no key“, die SKILL.md-Dateien sagen Key/OAuth – widersprüchlich). Bilder gehen an den three.ws-Server; wohin sie weitergereicht werden, steht nirgends.
- Reife: Skills-Repo mit 3 Commits, 0 Sternen, Lizenzangaben widersprüchlich (Apache-2.0 vs. MIT).
- **Urteil: Parken.** Für Lilly unbrauchbar (kein Vierbeiner-Rig), für Misters Gesicht aus Datenschutz- und Lizenzgründen nicht empfehlenswert. Höchstens für Deko-Props per Text.

**Scenario** – Plattform, die 20+ 3D-Modelle bündelt (u. a. Rodin Gen-2, Tripo 2.5 Multi-View mit 2–4 Bildern, Trellis 2, Meshy) und Tripo-Rigging als Modell führt. Ausgabe GLB.
- Laut Hilfecenter (Okt. 2026) sind etliche Modelle bereits **deprecated** (Hunyuan 3D 3.0/2.1, Tripo P1/3.0, Sparc3D, Meshy 7) → Katalog vor Nutzung prüfen.
- Kosten: Credits werden pro Generierung abgezogen; ein **Dry-Run-Parameter** liefert die exakten Kosten ohne Abbuchung (passt perfekt zur Freigabepflicht). Planpreise **nicht verifiziert** (scenario.com ist aus dieser Cloud-Session blockiert; Drittquelle: Einstieg ca. 15 $/Monat; API laut einer Quelle erst ab Pro).
- Stärke: alles an einem Ort, Charakter-Konsistenz für 2D-Ansichten (Seite/Rücken) – nützlich als Vorstufe.
- **Urteil: Testen – aber nur, wenn die Direktanbieter (Tripo/Meshy) nicht reichen.** Ein Abo nur für 3D lohnt sich nicht, solange ein Lilly-Modell bei Tripo unter 1 $ kostet.

**Luma** – Genie (Text/Bild→3D) ist laut mehreren Drittquellen seit Januar 2026 eingestellt; die Genie-Seite leitet auf die Startseite um, die Pläne umfassen nur noch Bild und Video. Eine offizielle Einstellungsmeldung habe ich nicht gefunden.
- **Urteil: Für 3D Killen.** Für die Shorts-Fabrik (Video) weiterhin denkbar.

### 3.2 Alternativen, die besser passen

| Werkzeug | Katze (Vierbeiner) | Mensch | Kosten | Haken | Urteil |
|---|---|---|---|---|---|
| **Meshy** | Bild→3D; Rigging bietet „Quadruped“ (Web-App) | ja | **belegt:** Free 100 Credits/Monat, Bild→3D 20 Credits, Rigging 0 Credits; Pro ca. 20 $/1.000 Credits (Drittquelle) | Vierbeiner-Animationsbibliothek dünn; Doku teils widersprüchlich; Meshy-7 als 3D-Modell läuft laut Doku am 31.12.2026 aus; Lizenz des Free-Tiers **ungeprüft** | **Kostenloser Testlauf zuerst** |
| **Tripo** | Bild→3D + Multi-View (2–4 Bilder); Rig-Typ `quadruped` ab Rigger v2.5; Presets u. a. Gehen/Laufen | ja (90+ Presets im v1.0-Rig) | **belegt (API):** 1 Credit = 0,01 $; Bild→Modell texturiert 30 Credits (≈0,30 $), Auto-Rig 25 (≈0,25 $), je Animation 10 (≈0,10 $); Rig-Check gratis | API- und Studio-Credits getrennt; Vierbeiner-Presets begrenzt | **Bezahlte Hauptroute für Lilly** |
| **Rodin (Hyper3D)** | Bild/Multi-View→3D, hohe Geometriequalität | ja | ca. 0,40 $/Generierung über fal.ai (Drittanbieter); offiziell 1,50 $/Credit, Basis 0,5 Credits | Kein Rigging-Angebot gefunden | Reserve, falls Tripo-Geometrie nicht reicht |
| **TRELLIS.2** (Open Source, MIT) | ja | ja | 0 € Software | braucht NVIDIA-GPU ≥ 24 GB | Läuft **nicht** auf Intel-Grafik |
| **Hunyuan3D 2.1** (Open Source) | ja | ja | 0 € Software | 10–29 GB VRAM; Lizenz schließt EU aus | Nicht nutzbar |
| **VRoid Studio** | – | stilisiert (Anime), lokal | 0 € | kein Foto-Import, Gesicht per Regler | **Mensch, Route A** |
| **Avaturn** | – | Foto→GLB, 52 ARKit-Blendshapes, Mixamo-kompatibel (Drittquellen) | API laut Verzeichnissen ab ca. 800 $/Monat (evtl. veraltet) | Foto geht in die Cloud | Mensch, Route B (nur nach Prüfung) |
| **Ready Player Me** | – | – | – | seit 31.01.2026 abgeschaltet | Raus |

## 4. Technische Kernerkenntnisse

1. **Ein Einzelbild reicht nicht für Treue.** Bild→3D erfindet Rücken und Seiten. Für Lilly erst konsistente Ansichten (vorne, Seite, hinten) erzeugen, dann Multi-View→3D (Tripo 2.5 Multi-View oder Rodin).
2. **Sitzpose vs. Auto-Rig.** Auto-Rigger erwarten eine stehende Neutralpose. Das Masterbild sitzt. Lösung in zwei Spuren:
   - *Dashboard-Lilly* (sitzend): leichtes Eigen-Rig in Blender mit ca. 25 Knochen (Wirbelsäule, Hals, Kopf, Kiefer, Ohren, Schwanzkette, Lider, Vorderbeine). Reicht für Atmen, Kopfneigen, Ohrenzucken, Schwanzwedeln, Pfote heben, Sprechen.
   - *Bewegungs-Lilly* (stehend, optional für Shorts): Tripo-Quadruped-Rig mit Gehen/Laufen.
3. **Gesicht kommt von keinem Auto-Rigger.** Blinzeln und Mund brauchen Shape Keys bzw. Kieferknochen – und die großen Augen sollten als **eigene glänzende Kugeln** modelliert werden (echte Glanzlichter, Blickrichtung, Lider). Das macht Lilly wiedererkennbar; aufgebackene Augen in der Textur wirken tot.
4. **Fritz braucht kein Skelett.** Ein Gehirn wird per Code animiert (Pulsieren, Synapsen-Glühen, Farbwechsel je Zustand). Prozedural in three.js = 0 €, sofort machbar, perfekt im Neon-Stil.
5. **Mensch über VRM.** Der VRM-Standard bringt Blinzeln und Mundformen (aa/ih/ou/ee/oh) bereits mit; `@pixiv/three-vrm` steuert sie in three.js. Damit kann der Audiopegel direkt den Mund bewegen.

## 5. Ressourcen-Budget (Startwerte für Intel-Grafik)

Eigene Zielwerte, **nicht gemessen** – werden in Phase 1 auf Misters Gerät kalibriert.

| Größe | Lilly | Fritz | Mensch |
|---|---|---|---|
| Dreiecke | ≤ 30.000 | ≤ 10.000 | ≤ 30.000 |
| Knochen | ≤ 64 | 0 | ≤ 64 |
| Textur max. | 1024 px | keine | 1024 px |
| GLB-Datei | ≤ 3 MB | – (Code) | ≤ 3 MB |

Renderer-Regeln: Pixelratio ≤ 1,5 · keine Echtzeit-Schatten (Blob-Schatten) · kein Post-Processing (der bestehende Canvas-Strahlenkranz bleibt hinter der Figur und liefert das Neon-Glühen) · max. 30 fps · Pause bei verstecktem Tab · automatische Rückkehr zum SVG-Kopf, wenn fps unter 24 fallen oder WebGL2 fehlt · Geometrie mit meshopt komprimieren (besser für animierte Modelle als Draco).

## 6. Empfehlung

| Figur | Route | Kosten bis zum ersten Ergebnis |
|---|---|---|
| **Lilly** | Masterbild → Meshy-Free-Test (bis zu 5 Versuche) → bei Bedarf Tripo Multi-View → Blender-Feinschliff + Leicht-Rig → Studio-Abnahme | 0 € (Free) bzw. ca. 1–5 € (Tripo, mit Freigabe) |
| **Fritz** | Prozedural in three.js | 0 € |
| **Mensch** | VRoid Studio → VRM → three-vrm | 0 € |
| **Studio** | `studio.html`: GLB/VRM per Drag-and-drop importieren, Zustände testen, Budget prüfen, Masterbild halbtransparent drüberlegen | 0 € |

**Kostenregel:** Vor jedem bezahlten Aufruf wird ein Angebot erstellt (Tripo-Preistabelle bzw. Scenario-Dry-Run), Mister bekommt Betrag und Zweck angezeigt, und erst nach ausdrücklicher Freigabe läuft der Auftrag. API-Keys bleiben im PC-Backend (`.env`), nie im Browser.

## 7. Offen / unbekannt

- Scenario-Planpreise und 3D-Credit-Kosten (Seite aus dieser Session blockiert) → per Dry-Run klären
- Lizenz der Meshy-Free-Tier-Ausgaben (öffentlich? CC BY?) → vor Nutzung als Marken-Maskottchen prüfen
- Intel-GPU-Modell und tatsächliche fps
- Ob HUD/Basis auf dem PC schon three.js nutzt (dann Version angleichen)
- Wer das Blender-Rig baut: per Python-Skript (Claude/AutoClaw) oder als Fiverr-Auftrag – Preis unbekannt

## Quellen

- three.ws: [Skills-Pack](https://github.com/nirholas/three-ws-3d-skills), [generate-3d-model](https://github.com/nirholas/three.ws/blob/main/.agents/skills/generate-3d-model/SKILL.md), [rig-a-model](https://github.com/nirholas/three.ws/blob/main/.agents/skills/rig-a-model/SKILL.md), [create-3d-avatar](https://github.com/nirholas/three.ws/blob/main/.agents/skills/create-3d-avatar/SKILL.md)
- Scenario: [3D-Features](https://www.scenario.com/features/3d), [Modellvergleich (Hilfe)](https://help.scenario.com/en/articles/comparing-generative-3d-models/), [API-Credits](https://help.scenario.com/en/articles/api-usage-and-credits-creative-units), [API-Preise](https://www.scenario.com/api-pricing), [Tripo-Rigging bei Scenario](https://app.scenario.com/api/models/model_tripo-rigging-v1/markdown)
- Luma: [Was aus Genie wurde](https://supavoxel.com/vs/luma-genie), [Genie 3D API](https://www.llmreference.com/provider/luma-api/genie-3d)
- Meshy: [Preise & Credits](https://docs.meshy.ai/en/webapp/pricing), [Rigging-API](https://docs.meshy.ai/en/api/rigging), [Rigging-Guide](https://docs.meshy.ai/webapp/guides/3d-model/rigging)
- Tripo: [API-Preise](https://developers.tripo3d.ai/en/pricing), [Rigging](https://developers.tripo3d.ai/en/models/rig), [Rust-Client rig_model](https://docs.rs/tripo-api/0.1.4/src/tripo_api/tasks/rig_model.rs.html)
- Rodin: [Hyper3D-Preise](https://hyper3d.ai/pricing), [fal.ai Rodin v2.5](https://fal.ai/models/fal-ai/hyper3d/rodin/v2.5/llms.txt)
- Open Source: [Hunyuan3D-2.1 vs TRELLIS.2](https://image3d.io/compare/hunyuan3d-2-1-vs-trellis-2-2026/)
- Avatare: [Ready Player Me eingestellt](https://avatarsdk.com/blog/2026/01/15/switch-from-ready-player-me-to-avatar-sdk-fast-familiar-production-ready/), [Avatar-Plattformen 2026](https://avatarsdk.com/blog/2026/08/31/avatar-platforms-2026-whos-alive-whos-gone/), [Avaturn (Verzeichnis)](https://similarlabs.com/p/avaturn), [VRM-Ausdrücke](https://nishlumi-vrmviewmdoc.readthedocs.io/en/latest/man2/prop_vrm.html)
- Performance: [glTF-Kompression Draco/meshopt/KTX2](https://jvinhit.github.io/blog/threejs-16-gltf-compression-draco-meshopt-ktx2/), [Auto-Rigging-Vergleich 2026](https://ludo.ai/compare/best-ai-auto-rigging-tools)
