# fussball-vereine

Fußball-Apps für luxemburgische Vereine, eine je Verein, alle aus derselben Vorlage: nächste Spiele,
Tabelle, Ergebnisse, alle Spiele der Klasse, Heute-Spieltag, Kalender-Abo je Mannschaft, BGL und
Roude Léiw, App teilen, **Liveticker für Seniors 1 und das Nationalteam**. Daten von der FLF (und
FuPa für Ticker, BGL-Tabelle, Nationalteam), automatisch aktuell.

Entstanden aus der App „FC Koeppchen“ (`fussball.html` im Repo `panini-wm-2026`). **Die bleibt, wo
sie ist, und wird hier weder gelesen noch geändert** — dieses Repo ist eine eigene Kopie (Stand
19.10.2026). Verbesserungen dort wandern nicht von selbst hierher.

> **Stand 25.10.2026: ruht.** Kein Verein eingetragen (die Yellow Boys waren nur zum Zeigen und sind
> gelöscht), GitHubs Zeitplan im Workflow ist auskommentiert, der Worker-Cron bei Cloudflare aus.
> Für einen neuen Verein: eintragen wie unten, `schedule:` in `.github/workflows/fussball.yml` wieder
> einkommentieren, Cron-Trigger beim Worker wieder anlegen.

Ein Repo, ein Workflow, ein Worker für alle Vereine. App eines Vereins:
`https://strakekine.github.io/fussball-vereine/<ordner>/`

## Neuer Verein

Gebraucht: Vereinsname, **Logo** (PNG/JPG/SVG, quadratisch am schönsten), **zwei Farben** (`kopf` für
die Kopfleiste, `akzent` für Linie, Siege, Knöpfe). Rest findet die Suche.

```sh
export NODE_USE_ENV_PROXY=1               # nur in der Cloud-Umgebung nötig
node tools/suche.mjs bertrange            # → 500009  Sporting Bertrange
node tools/suche.mjs 500009               # Mannschaften, FuPa-Name, Vorschlag für verein.json
# Vorschlag als Datei speichern, name/kurz/farben anpassen
node tools/neu.mjs verein.json logo.png --abruf
git add -A && git commit -m "Neuer Verein: …" && git push
```

`neu.mjs` legt `<slug>/` an (verein.json, logo.png, index.html), schreibt `vereine.json` (die Liste
für den Worker), lässt den Test laufen und holt mit `--abruf` gleich die ersten Daten. Danach kommt
der Verein bei jedem Lauf von selbst mit.

Nach einer Änderung an `vorlage/index.html`, `tools/neu.mjs` oder `repo.json`: `node tools/neu.mjs
--alle` baut alle Vereine neu (Fassung zählt hoch). Der Test merkt, wenn das vergessen wurde.

## verein.json

| Feld | Bedeutung |
|---|---|
| `name` | App-Name, Titel, Home-Bildschirm („US Mondorf“) |
| `kurz` | statt des FLF-Namens in Listen und im Kalender („Mondorf U13 1“) |
| `flfName` | **wie die FLF die Mannschaften schreibt**, ohne I/II — nicht wie den Verein (Koeppchen: Verein „Wormeldange FC Koeppchen“, Mannschaften „FC Koeppchen Wormeldange I“). Die Suche schlägt ihn vor |
| `clubId` | FLF-Vereinsnummer |
| `slug` | Ordner und Adresse (`mondorf`). Kalender-UIDs und Speicher hängen daran: **nicht mehr ändern**, sobald die App benutzt wird |
| `fupa` | FuPa-Vereinsslug für den Liveticker (`us-mondorf`, `sporting-bartringen`). Leer = kein Ticker |
| `farben` | `{ "kopf": "#rrggbb", "akzent": "#rrggbb" }` — Schrift, helle Töne, Aufstieg/Niederlage werden daraus gerechnet und auf Lesbarkeit geprüft |
| `start` | Reiter beim ersten Öffnen (`"U13 1"`, `"Seniors 1"`; Standard U13 1, sonst der erste) |
| `von` | freiwillig: „Gemacht von …“ unten in der App, sonst der aus `repo.json` (leer = weg). **Ohne Logo** — das Koeppchen-Logo darf bei fremden Vereinen nicht stehen (keine Erlaubnis des Vereins) |
| `alter` | freiwillig: Altersklassen und Reihenfolge als FLF-ids. Standard Seniors, U19, U17, U15, U13, U11, U9, Dames, Réserves (`[1,2,3,11,4,7,5,17,9999]`); alle in `tools/fussball.mjs` (`ALLE_AGES`) |
| `bgl`, `nati` | Reiter BGL und Roude Léiw (Standard an) |
| `liveReiter` | freiwillig: Reiter mit Ticker, Standard `"Seniors 1"` |
| `teilen` | freiwillig: eigener Text für „App teilen“ (Link selbst hineinschreiben) |

`repo.json` (eine für alle): `repo` (`konto/name`), `worker` (Adresse des Workers, leer = ohne) und
`von` („Laurent R.“, steht unten in jeder App, nur der Name).

## Einmal einrichten

1. **GitHub**: Repo `fussball-vereine` anlegen (öffentlich — Pages und Actions kosten dann nichts),
   diesen Inhalt hochladen. Settings → Pages: „Deploy from a branch“, `main`, `/ (root)`.
   Settings → Actions → General → Workflow permissions: „Read and write“.
2. **Cloudflare** (für Liveticker und pünktlichen Abruf): `tools/worker.js` als Worker einfügen,
   Anleitung oben in der Datei. Secret `GITHUB_TOKEN` (Fine-grained, nur dieses Repo, Actions: Read
   and write, **läuft nach höchstens einem Jahr ab**), Cron `*/15 * * * *`. Adresse in `repo.json`
   als `worker`, dann `node tools/neu.mjs --alle`.

Ein Worker für alle: Er liest die Vereine aus `vereine.json`, ein neuer Verein braucht dort nichts.

## Wie es läuft

- **Abruf** (`.github/workflows/fussball.yml` → `tools/fussball.mjs`): alle Ordner mit verein.json
  nacheinander, je Verein gut eine Minute. BGL und Nationalteam einmal für alle (`gemeinsam.json`,
  dort auch das Namensgedächtnis der BGL-Tabelle). Fällt ein Verein aus, behält er seinen alten
  Stand und die anderen laufen weiter. Commit nur bei Änderung.
- **Takt** (Luxemburger Zeit, der Worker rechnet selbst um): jeden Tag 7:00 und 23:30, Mo–Fr
  zusätzlich 22:15, Sa/So 8–22 Uhr jede Viertelstunde; GitHubs eigener Zeitplan als Reserve (oft
  verspätet). Ohne Worker nur die Reserve. Die Fußzeile der App nennt dieselben Zeiten.
- **Liveticker**: nur über den Worker, nur Seniors 1 (und Nationalteam). Die App fragt ihn ab 15 min
  vor Anstoß bis 3 h danach alle 20 s (`?verein=…&anstoss=…`); er sucht das Spiel bei FuPa
  (`<fupa>-m1-<saison>`) und gibt Stand, Minute, Tore, Karten zurück. Der LIVE-Kasten kommt erst,
  wenn FuPa wirklich etwas hat (Stand, Tickerer oder Tor) — FuPa setzt ab Anpfiff von selbst
  „LIVE“. Vorher steht im Spieltag-Kasten „Liveticker, wenn verfügbar …“, ab Anpfiff „falls bei FuPa
  jemand tickert“, ab 15 min ohne Tickerer „Bisher tickert niemand bei FuPa …“. **Leer, wenn bei FuPa
  niemand am Platz tickert** — das liegt am Verein. Führt FuPa Heim/Gast andersherum (der Slug
  beginnt mit der Heimmannschaft), werden Stand und Tore auf die FLF-Reihenfolge gedreht.
- **Countdown**: 15 min vor Anpfiff „Gleich geht's los! ⚽ · Anpfiff in n Minuten“ (nicht bei
  Uhrzeit offen).
- **Unsere Woche**: breiter Knopf über den Reitern (Vereinslogo links und rechts, „· n Spiele“,
  „· heute n Spiele“ oder „· 🔴 n Spiele gerade LIVE“). Alle Spiele des Vereins Mo–So (ab So 22 Uhr
  die nächste Woche), je Spiel einmal, Schild der Mannschaft, 🏠/✈️, Uhrzeit oder „läuft“/„🔴 LIVE“,
  📍 bis zum Ergebnis, „gleichzeitig mit …“, nach dem Wochenende die Bilanz. Tippen öffnet den Reiter.
- **Stadion** (📍) bei allen kommenden Spielen, auch im BGL-Reiter; Forfait mit Termin in der
  Zukunft heißt „Forfait, im Voraus gewertet“. Ohne Worker kein Ticker; das Ergebnis kommt mit dem nächsten FLF-Abruf. Anders als bei
  Koeppchen gibt es keinen Rückfall über GitHub (`live.json`): der bräuchte für viele Vereine
  gleichzeitig laufende Wachen.
- **Spielgemeinschaften** (`ENT Mondorf/Schengen I`) zählen mit, die FLF führt sie beim Verein. Die
  Reiter-Nummer kommt aus dem I/II/III am Ende („U13 1“); eine einzige Réserves ohne I/II heißt
  „Seniors 2“.
- **Speicher**: alle Apps liegen auf derselben Herkunft (`strakekine.github.io`), darum je Verein
  ein eigener Schlüssel (`fussball_<slug>`) — sonst sähe eine App den Stand einer anderen.
- **Auf-/Abstieg** (`REGELN` in `vorlage/index.html`) ist das FLF-Rundschreiben für U13/U15, gilt für
  alle Vereine. Neues Rundschreiben → dort nachtragen, `--alle`.

## Aus der Koeppchen-App übernommen

Stand der Vorlage: Koeppchen bis Commit `9081a07` (20.10.2026, Fassung k), am 20.10. nachgezogen.
**Bewusst nicht übernommen**: Liveticker für Seniors 2/3 (bei Koeppchen über FuPa-Slugs, die der
Abruf vorab holt) und der anonyme Zähler (D1 im Worker). Neue Koeppchen-Änderungen kommen nicht von
selbst — nachziehen per Drei-Wege-Abgleich (Basis = `fussball.html` beim letzten Abgleich).

## Testen

`node test/fussball.mjs` — Abruf-Aufbereitung, App-Logik mit Beispielverein, Farben, Worker (mit
nachgebautem Netz: Verein aus vereine.json, Spiel per Anstoß, nur eigene Slugs, 20 s gemerkt) und
für jeden eingetragenen Verein: gebaut, aktuell, Logo da. Läuft vor jedem Abruf in der Action.
Gegenprobe: Worker ohne Slug-Prüfung → 3 Fehler.

Geprüft am 19.10.2026 mit Mondorf und Bertrange (echte FLF-Daten), Gegenprobe mit Koeppchen
(dieselben Daten wie `data/fussball.json` dort), Worker gegen echtes FuPa (Mondorf–Wiltz 4:1 mit
allen Torschützen über die FLF-Anstoßzeit gefunden).
