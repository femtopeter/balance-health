# Balance — Gesundheits-PWA

Ganzheitliche Gesundheits-App als Progressive Web App fürs iPhone.
Vier Säulen: **Training · Ernährung · Erholung/Schlaf · Arbeit** — evidenzbasiert abgestimmt.

**Live:** https://femtopeter.github.io/balance-health/

> 📋 Weiterentwicklung? → **[HANDOFF.md](HANDOFF.md)** (Roadmap, Evidenzbasis, Architektur, Fallstricke)

---

## Features

**Heute** — Balance-Ring über alle vier Säulen, Quick-Tiles, evidenzbasierte Nudges
**Training** — 6-Wochen-Mesozyklus (3× Progressive → Overreaching → Peak → Deload), Rep-Logging mit Ziel-Vergleich, kg-Tracking bei Hantelübungen, Trend-Sparklines, Muskelgruppen-Statistik
**Ernährung** — personalisiertes Proteinziel (g/kg), Makros, Schlaf, Alkohol-Tracking, 7-Tage-Durchschnitte
**Erholung & Arbeit** — Resonanzatmung (5.5/min, animiert), Stress & Stimmung, Arbeitszeit + Mikropausen, Hike-&-Fly-/Zone-2-Log
**Profil** — Zielwerte anpassbar, JSON-Backup Export/Import, dokumentierte Studienlage

Dazu: Dark Mode, offline-fähig, **alle Daten bleiben auf dem Gerät** (localStorage, kein Server).

## Auf dem iPhone installieren

1. URL in **Safari** öffnen
2. **Teilen** → **„Zum Home-Bildschirm"**

Läuft dann wie eine native App — Vollbild, offline, eigenes Icon.

## Dateien

```
.
├── index.html      # Komplette App (single file, Vanilla JS)
├── manifest.json   # PWA-Manifest
├── sw.js           # Service Worker (offline)
├── icons/
├── HANDOFF.md      # Roadmap & Doku für Weiterentwicklung
└── README.md
```

## Deployment

GitHub Pages, Source: `main` / `/root`. Push genügt:

```bash
git add -A && git commit -m "…" && git push
```

Pages baut in ~1 Minute. Bei alter Version am iPhone: PWA vom Homescreen löschen und neu hinzufügen — der Service Worker cached aggressiv.

> Pfade müssen **relativ** (`./`) bleiben — absolute Pfade (`/index.html`) brechen auf der Pages-Unterordner-URL.
