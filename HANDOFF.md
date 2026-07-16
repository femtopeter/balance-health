# Balance — Handoff & Roadmap

> **Für die nächste Session:** Lies dieses Dokument. Die Roadmap (Prio 1–3) ist **komplett umgesetzt
> und deployed** (Juli 2026). Als Nächstes: offene Punkte in Abschnitt 9, oder Strava (Abschnitt 4).
> Alles Nötige steht hier — Recherche ist bereits gemacht, bitte nicht wiederholen.
>
> **Offen (auf Nassims Seite):**
> - Bargella-Koordinaten (er hatte zweimal Grabs kopiert) — im Profil-Tab als Wetter-Ort nachtragbar.
> - Kalender: einmalige Azure-App-Registrierung + erstes Anmelden (Anleitung im Profil-Tab der App).

---

## 1. Was das ist

**Balance** — ganzheitliche Gesundheits-PWA für Nassim. Vier Säulen: **Training · Ernährung · Erholung/Schlaf · Arbeit**.
Entstanden aus einem reinen Trainings-Tracker, in v3 zur Gesundheits-App erweitert.

- **Live:** https://femtopeter.github.io/balance-health/
- **Repo:** https://github.com/femtopeter/balance-health (public, GitHub Pages: `main` / `/root`)
- **Nutzung:** iPhone, Safari → „Zum Home-Bildschirm"

---

## 2. Architektur (bewusst simpel — bitte beibehalten)

- **Vanilla JS, Single-File** `index.html` (~1200 Zeilen). Kein Framework, kein Build-Step.
- **localStorage**, Key `training-v3` (Auto-Migration von `training-v2` in `loadState()`).
- **PWA:** `sw.js` (Cache `balance-v3`) + `manifest.json`. Pfade **relativ** (`./`) — zwingend für Pages-Unterordner-URL.
- **Dark Mode** via `prefers-color-scheme`.
- **Kein Backend.** Daten bleiben auf dem Gerät.

### Datenmodell
```json
{
  "version": 3,
  "week": 1,
  "logs":    { "W1-Mi-Goblet Squats": [9, "", "", ""] },
  "weights": { "W1-Mi-Goblet Squats": 20 },
  "nutrition": { "2026-07-15": { "sleep":7.5, "sleepQuality":4, "protein":130,
                   "carbs":200, "fat":65, "calories":2100, "alcohol":0, "notes":"" } },
  "recovery":  { "2026-07-15": { "stress":2, "mood":4, "breathMin":3 } },
  "work":      { "2026-07-15": { "hours":8.5, "breaks":4 } },
  "hikefly":   [ { "date":"2026-07-15", "ascent":850, "dur":95, "loc":"Grabs" } ],
  "profile":   { "bodyweight":75, "proteinPerKg":1.8, "sleepTarget":8,
                 "calorieTarget":2500, "breathMinTarget":5, "workDayTarget":8.5, "breaksTarget":4 },
  "morningOpen": { "morning-0": true }
}
```
Alle Zielwerte leiten sich aus `profile` ab (Proteinziel = `bodyweight × proteinPerKg`).

### Wichtige Funktionen
| Funktion | Zweck |
|---|---|
| `loadState()` | State laden + v2→v3 Migration |
| `pillarScores()` | 0–100 je Säule → Balance-Ring auf „Heute" |
| `recommendations()` | **Empfehlungs-Engine**: regelbasiert, priorisiert, max. 3 Empfehlungen mit «Warum» |
| `renderNudges()` | Rendert Engine-Output + kompakte Reminder-Zeile auf «Heute» |
| `sparkline(wKey)` | Inline-SVG-Trend, bester Satz je Woche |
| `startBreath()` / `stopBreath()` | Resonanzatmung 5.5/min |
| `exportData()` / `importData()` | JSON-Backup |

---

## 3. Roadmap — in dieser Reihenfolge (vom Nutzer bestätigt)

### ✅ Prio 1 — Empfehlungs-Engine  — **ERLEDIGT (Juli 2026)**

Umgesetzt als `recommendations()` in `index.html` (regelbasiert, priorisiert, max. 3, jede Empfehlung mit aufklappbarem «Warum» inkl. Quellen). Abgedeckt: hart/locker (Schlaf, Stress, Deload), Abend-Session streichen (morgens erledigt + Schlafmangel), Überlastungs-Marker (≥2 von Schlaf/Stress/Stimmung über 3 Tage), Alkohol gestern/heute, Protein- & Kalorien-Lücke, «Grünes Licht»-Freigabe, H&F-Fenster am AKTIV-Tag (nur Erholungssicht, **kein** Flug-Verdikt), Daten-fehlen-Hinweis. Woche 4 wird als geplantes Overreaching erklärt. SW-Cache auf `balance-v4`. Getestet über localhost mit 7 Szenarien.

**Ursprüngliches Ziel (Referenz):** Aus den bereits geloggten Daten konkrete Tagesempfehlungen ableiten. Ersetzt/erweitert `renderNudges()`.

**Warum zuerst:** Braucht keinen neuen Datenzugang, liefert sofort Wert, und wird in *jedem* Szenario gebraucht — egal woher die Daten später kommen. Erst Infrastruktur bauen und dann merken, dass die Empfehlungen nicht taugen, wäre die teure Reihenfolge.

**Verfügbare Inputs:** `nutrition` (protein, calories, sleep, sleepQuality, alcohol), `recovery` (stress, mood, breathMin), `work` (hours, breaks), `logs`/`weights`, `hikefly`, `profile`.

**Die 5 Entscheidungen, die die Engine treffen soll:**
1. Morgens oder abends trainieren?
2. Hart oder locker heute?
3. Hike & Fly — Fenster heute? *(braucht Wetter → Prio 2)*
4. Wie viel essen?
5. Überreiche ich gerade?

**Regelvorschläge (evidenzbasiert, Schwellen aus Abschnitt 5):**
- Schlaf < 7h (3-Tage-Ø) → heute Volumen/Intensität runter, keine harte Session
- Protein < 80 % Ziel (3-Tage-Ø) → Recovery limitiert, Hinweis mit `proteinPerMeal()`
- Stress ≥ 4 **und** Schlaf < 7h → Technik/Mobility statt Last, Atmung vorschlagen
- Alkohol gestern + harte Session heute → Erwartung dämpfen (MPS −24–37 %)
- Woche 6 → Deload-Guidance
- Volumeneinbruch außerhalb Deload → mögliches Overreaching
- Zwei Einheiten/Tag: Morgens erledigt + schlechter Schlaf → Abend-Session streichen oder auf Mobility

**Designprinzipien:**
- **Regelbasiert und transparent.** Immer das *Warum* mitliefern (Nassim ist Wissenschaftler — er will die Begründung). Kein ML, keine Blackbox.
- Priorisieren: max. 2–3 Empfehlungen, nicht zehn Nudges.
- Keine medizinischen Aussagen. Trainingssteuerung ≠ Diagnostik.

---

### ✅ Prio 2 — Wetter — **ERLEDIGT (Juli 2026)**

Umgesetzt mit [Open-Meteo](https://open-meteo.com/) (keine Auth, CORS-frei): Wetter-Karte auf «Heute» mit
10-Stunden-Vorschau ab jetzt — Temperatur, Wind 10 m (Richtung + km/h), Böen, Höhenwind ~1500 m (850 hPa),
Regen (mm/h), Bewölkung. 30-min-Cache in localStorage (separate `wx-*`-Keys, nicht im Backup), Offline-Fallback
auf letzte Daten mit Zeitstempel. Orte editierbar im Profil-Tab (`state.locations`), Umschalt-Buttons bei >1 Ort.
Funktionen: `fetchWeather()`, `refreshWeather()`, `renderWeatherCard()`. SW-Cache auf `balance-v5`.

- **Grabs:** 47.1666 N, 9.4242 E (von Nassim bestätigt, Google Maps).
- **Bargella: Koordinaten stehen noch aus** — er hatte versehentlich zweimal Grabs kopiert. Im Profil-Tab
  als Ort ergänzen, sobald vorhanden.
- **Zweck:** Zusammen mit dem Kalender ist das Nassims wörtliche Entscheidungsregel: *„wenn es das Wetter und die Meetings zulassen"*.

> ⚠️ **Sicherheitskritisch:** Die App darf **kein Go/No-Go fürs Fliegen** aussprechen. Gleitschirm ist sicherheitsrelevant; eine Fehlentscheidung kann tödlich sein. Nur **rohe Bedingungen anzeigen**, die Beurteilung bleibt beim Piloten. Keine „heute fliegbar"-Verdikte.

---

### ✅ Prio 3 — Outlook Kalender — **CLIENT-SEITIG FERTIG (Juli 2026)**

Umgesetzt ohne MSAL: handgeschriebener **Auth-Code-Flow mit PKCE (S256)** direkt in `index.html`
(`calLogin()`, `calHandleRedirect()`, `calAccessToken()`). Graph-Call: `/me/calendarView` mit
`$select=start,end,showAs,isAllDay` — **nur Frei/Belegt, keine Betreffzeilen, keine Inhalte**.
Karte «Kalender heute» auf dem Heute-Tab: Timeline 06–21 Uhr (belegt/frei) + Liste der freien Fenster.
Engine-Regel: größtes freies Fenster 08–19 Uhr ≥3h → Hinweis «Freies Fenster» (Kalender-Hälfte von
Nassims Regel *„wenn Wetter und Meetings es zulassen"* — Wetter-Rohdaten stehen daneben, kein Verdikt).

**Speicherung:** `cal-cfg` (Client-ID, Tenant), `cal-tok` (Tokens), `cal-cache` (15-min-Cache) —
eigene localStorage-Keys, **bewusst nicht im State** → landen nie im JSON-Backup.

**Was Nassim noch tun muss (Anleitung im Profil-Tab, aufklappbar):**
1. App-Registrierung auf portal.azure.com (SPA-Plattform, Redirect-URI = exakte Pages-URL, Delegated `Calendars.Read`).
2. Client-ID im Profil-Tab eintragen, anmelden, Consent bestätigen.

**Status Tenant:** Nassim sagt (Juli 2026), der Firmen-Tenant erlaubt private iPhone-Apps / User-Consent —
sollte also ohne IT gehen. Falls die Registrierung im Firmen-Tenant doch gesperrt ist: Multi-Tenant-App
über ein privates (kostenloses) Azure-Konto registrieren und mit dem Arbeitsaccount konsentieren.

**Bekannte Grenze:** SPA-Refresh-Tokens leben ~24h → ~tägliches Neu-Anmelden (ein Tap bei bestehender
Microsoft-Session). Echte Dauer-Verbindung ginge erst mit Backend (Beelink, sobald angeschafft).

---

## 4. Bereits geprüft — NICHT nochmal recherchieren

| Quelle | Befund |
|---|---|
| **WhatsApp** | ❌ **Endgültig nein.** Keine offizielle API für persönliche Chats. Business-API ist Firma→Kunde; eine Nummer kann **nicht gleichzeitig** in der App und an der API hängen. Drittanbieter klinken sich per QR in die WhatsApp-Web-Session — außerhalb Metas Programm, Sperr-Risiko, plus Nachrichten Dritter ohne deren Einwilligung. Nutzen ohnehin minimal. |
| **Apple Health / Watch** | ❌ Kein Web-Zugriff. HealthKit ist für Safari/PWAs nicht zugänglich. Einzige Wege: **Kurzbefehle (Shortcuts)** exportieren JSON, oder native App. |
| **Coop Supercard** | ❌ Nur In-App-Ansicht (~1 Jahr), kein Export. Nur via rechtliches Auskunftsbegehren — nichts für Routine. |
| **Lidl Plus** | ❌ Gleiches Bild, nur in-App. |
| **Migros** | ⚠️ CSV-Export existiert: `account.migros.ch/purchases/receipts`. Offizielle API nur Migros-intern. **Aber:** Bons sagen, was *gekauft*, nicht was *gegessen* wurde — und enthalten keine Nährwerte (Mapping via Open Food Facts, bei Schweizer Eigenmarken löchrig). Taugt für **Trends**, nicht fürs Tagesprotein. |
| **Outlook Mail** | ⚠️ Technisch möglich (Graph `Mail.Read`), **bewusst ausgeschlossen**: riesiges Volumen, kaum entscheidungsrelevantes Signal, maximales Risiko (Arbeitsaccount). |
| **Strava** | ✅ Einzige echte offizielle API für Trainingslast + Hike & Flys. Guter Kandidat nach Prio 3. |

**Grundsatz:** Zugangsdaten werden nicht angefasst. Keine Logins in der PWA.

**Kernprinzip für alle künftigen Integrationen:** *Der Engpass ist nicht Datenbreite, sondern Entscheidungsrelevanz.* Vor jeder Integration fragen: **Welche der 5 Entscheidungen ändert sie?** Wenn keine → nicht bauen.

---

## 5. Evidenzbasis (recherchiert — Zahlen direkt verwendbar)

- **Protein:** 1.6–2.2 g/kg/Tag; ~0.4 g/kg auf ≥4 Mahlzeiten. Proteinreiches **Frühstück** schlägt protein-lastiges Abendessen. *(ISSN Position Stand 2017; Schoenfeld & Aragon 2018; Yasuda 2020)*
- **Volumen:** 12–20 Sätze/Muskel/Woche; mehr Volumen = mehr Zuwachs, aber abnehmender Grenznutzen. Frequenz hilft v.a. der **Maximalkraft**. *(Pelland 2025; Schoenfeld 2017; Baz-Valle 2022)*
- **Schlaf:** ≥7h. Darunter über ≥14 Tage ≈ **1.7× Verletzungsrisiko**. *(Charest 2022; Rygielski 2024)*
- **Alkohol:** senkt Muskelproteinsynthese um **24–37 %**, auch mit Protein. Unter ~0.5 g/kg meist unkritisch. *(Parr 2014; Barnes 2014)*
- **Atmung:** ~0.1 Hz (5–6/min) erhöht HRV & Vagustonus. *(Laborde 2022, Meta-Analyse über 223 Studien)*
- **Mikropausen:** steigern Vigor, senken Fatigue; längere Pausen wirken stärker. *(Albulescu 2022, Meta-Analyse)*
- **Deload:** alle 4–6 Wochen, 5–7 Tage, Volumen/Intensität runter. *(Bell 2025; Rogerson 2024)*
- **Zwei Einheiten/Tag:** mehr Unterkörperkraft (Squat +16 % vs. +8 %), Hypertrophie ähnlich. *(Corrêa 2021)*

Volle Quellenliste mit Links: siehe „Wissenschaftliche Basis" im **Profil-Tab** der App.

---

## 6. Der Nutzer

Nassim, R&D Scientist, arbeitet 40–50 h/Woche, kocht möglichst selbst. **Trainiert meist zweimal täglich** (morgens vor + abends nach der Arbeit), abwechselnd Oberkörper/Beine, ~3 Sätze. **Di:** nur CrossFit morgens. **Spontane Hike & Flys** (Gleitschirm) in **Grabs oder Bargella**, wenn Wetter + Meetings es zulassen. **Wochenende:** Touren bis 3000 Hm oder Ausgehen mit Freunden.

Equipment: Pull-Up-Bar, Ringe, 2×14 kg Dumbbells (Upgrade auf verstellbare bis 32 kg geplant). Ziel: Plateau durchbrechen via Periodisierung + progressive Überlastung.

**Kommuniziert auf Deutsch. Will Begründungen, keine Behauptungen.** Offen für Effizienz-Vorschläge. Ein **Beelink Mini-PC** ist **noch nicht vorhanden** — die Anschaffung ist für die Zukunft geplant (Stand Juli 2026). Bis dahin: kein Backend möglich, alles bleibt client-seitig.

---

## 7. Lokales Testen — Fallstricke (Zeit sparen!)

- **Kein echtes Python/Node auf dem Rechner.** `python.exe` ist nur der Windows-Store-Stub und schlägt fehl.
- **Lösung:** PowerShell-`HttpListener`-Server. Skript-Vorlage in Abschnitt 8.
- **Umlaut-Falle:** Der Pfad enthält `Persönlich`. PowerShell 5.1 liest BOM-lose UTF-8-Skripte als ANSI → Pfad kaputt. Umlaut per `[char]0xF6` zusammensetzen:
  `"C:\Users\nassi\OneDrive\04_Pers" + [char]0xF6 + "nlich\..."`
- **Screenshots im Browser-Pane timen aus.** Stattdessen `read_page` (Accessibility-Baum) und `javascript_tool` nutzen — funktioniert zuverlässig.
- **`file://` geht nicht** für automatisierte Lesetools → immer über localhost testen.
- **Re-Render:** `logSet()` ruft `render()` → Element-Refs werden stale. Normal, kein Bug.

### Server-Snippet
```powershell
$root = "C:\Users\nassi\OneDrive\04_Pers" + [char]0xF6 + "nlich\Gesundheit\training-pwa\pwa"
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:8091/")
$listener.Start()
$mimes = @{ ".html"="text/html; charset=utf-8"; ".js"="application/javascript"; ".json"="application/json"; ".png"="image/png" }
while ($listener.IsListening) {
  try {
    $ctx = $listener.GetContext()
    $path = [System.Uri]::UnescapeDataString($ctx.Request.Url.AbsolutePath)
    if ($path -eq "/") { $path = "/index.html" }
    $file = Join-Path $root ($path.TrimStart("/"))
    if (Test-Path -LiteralPath $file -PathType Leaf) {
      $ext = [System.IO.Path]::GetExtension($file)
      $ctype = $mimes[$ext]; if (-not $ctype) { $ctype = "application/octet-stream" }
      $bytes = [System.IO.File]::ReadAllBytes($file)
      $ctx.Response.ContentType = $ctype
      $ctx.Response.OutputStream.Write($bytes, 0, $bytes.Length)
    } else { $ctx.Response.StatusCode = 404 }
    $ctx.Response.Close()
  } catch { }
}
```

---

## 8. Deployment

Git-Repo liegt **im `pwa`-Ordner** (`index.html` im Root — so will es Pages). Remote `origin` ist gesetzt, Credential Manager ist authentifiziert.

```powershell
cd "C:\Users\nassi\OneDrive\04_Persönlich\Gesundheit\training-pwa\pwa"
git add -A; git commit -m "…"; git push
```
Pages baut automatisch (~1 Min). Bei alter Version am iPhone: PWA vom Homescreen löschen und neu hinzufügen — der Service Worker cached aggressiv.

---

## 9. Offene Punkte (nach der Roadmap)

- [ ] **Mesozyklus-Abschluss**: Auswertung Ende Woche 6 — was hat sich verbessert, wo Stagnation
- [ ] **Richtiges App-Icon** (aktuell programmatisch generierter Platzhalter)
- [ ] **Feintuning nach iPhone-Test**: Touch-Größen, Lesbarkeit unterwegs
- [ ] **Vernetzte Waage** → Gewicht automatisch ins Proteinziel (aktuell der einzige manuell gepflegte Profilwert)
- [ ] Strava-Integration (nach Prio 3)
