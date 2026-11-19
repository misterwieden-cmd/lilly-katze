# Ideen & Vorhaben (Stand 2026-11-19)

Gesammelt aus der Diskussion mit Gemini + Claude. Nicht umgesetzt, nur gemerkt für später.

## Die 5 Ideen, bewertet

1. **Lillys Markt (Escrow)** — Finger weg bei echtem Geld (BaFin/ZAG-Thema). Als internes Spiel mit Fake-Credits harmlos.
2. **Voice- & Agenten-Router** — Stärkste Idee, Fundament. Phase-2-Router (Kumpel + Katze) ist schon der Anfang. Lokale Modelle über Ollama sparen Kosten (PC-Leistung noch ungeklärt).
3. **Nischen-Scout** — Zu vage ("Studien, Objekte oder Preise" = 3 Produkte). Erst auf eins festlegen.
4. **Echo of Little Lilly** — Passt zu YouTube Shorts, Nutzen ist Reichweite nicht Funktion. Günstig, gut für Viral-Plan.
5. **Audit & Fix-Worker** — Konkretest, am leichtesten zu bauen (Ladezeit, Seitengröße, TTFB messbar). Beste Geldidee, später als Dienst verkaufbar.

## Vorschlag Reihenfolge

- **Phase A — Router festigen (Fundament):** Intent-Erkennung (Kumpel/Katze/Scam-Check trennen), Fast/Deep an echte Modellwahl koppeln (Ollama lokal / Cloud deep), Fallback wenn Ollama down ist.
- **Phase B — Echo of Little Lilly:** Kurzform-Skript-Generator für Shorts (30-60s, Hook + Text), Produktion extern.
- **Phase C — Audit & Fix-Worker:** URL → Ladezeit/Größe/TTFB/kaputte Links → Report, später Fix-Vorschläge. Verkaufbares Modul.

Scam-Check aus Phase 1 bleibt unverändert als Katzen-Skill im Router.

## Offene Fragen für nächstes Mal

- Läuft Ollama schon auf dem Rechner? (PC-Leistung dafür ungeklärt)
- Backend/Router-Code liegt nicht in diesem Repo (nur Frontend-PWA) — muss für Phase A separat angebunden werden (eigenes Repo oder Remote-Control-Zugriff auf den Rechner).
