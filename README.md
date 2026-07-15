# Trainingsprogramm PWA

Fitness Tracker als Progressive Web App für iPhone.

## Deployment (GitHub Pages — gratis)

### 1. Repository erstellen
```bash
# Neues Git-Repo erstellen
cd pwa
git init
git add .
git commit -m "initial"
```

### 2. Auf GitHub pushen
```bash
# Auf github.com neues Repo erstellen: "training-app" (public)
git remote add origin https://github.com/DEIN-USERNAME/training-app.git
git branch -M main
git push -u origin main
```

### 3. GitHub Pages aktivieren
- Repo → Settings → Pages
- Source: "Deploy from a branch"
- Branch: `main`, Ordner: `/ (root)`
- Save

Die App ist in ~1 Minute live unter:
`https://DEIN-USERNAME.github.io/training-app/`

### 4. Auf iPhone installieren
1. Safari öffnen → URL eingeben
2. Share-Button (Quadrat mit Pfeil) tippen
3. **"Zum Home-Bildschirm"** wählen
4. Name bestätigen → Fertig

Die App läuft jetzt wie eine native App — Vollbild, offline-fähig, mit eigenem Icon.

## Alternative: Cloudflare Pages
```bash
# Wenn du Cloudflare nutzt:
npx wrangler pages deploy . --project-name=training-app
```

## Alternative: Vercel
```bash
npx vercel --prod
```

## Dateien
```
pwa/
├── index.html      # Komplette App (single file)
├── manifest.json   # PWA-Manifest
├── sw.js           # Service Worker (offline)
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
└── README.md
```

## Features
- 6-Wochen Mesozyklus (3P → OR → Peak → Deload)
- Rep-Logging pro Set mit Target-Vergleich
- Muskelgruppen-Statistiken
- Schlaf + Makro-Tracking mit 7-Tage-Durchschnitt
- Smarte Nudges (Protein, Schlaf, fehlende Logs)
- Offline-fähig
- Daten in localStorage (auf dem Gerät, kein Server)
