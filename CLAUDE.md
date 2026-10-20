# fussball-vereine

Fußball-Apps für luxemburgische Vereine (FLF), eine je Ordner, alle aus `vorlage/index.html`.
Nutzer: Laurent R. (Luxemburg), Deutsch, Du-Form, Widerspruch erwünscht. Entstanden am 19.10.2026
aus der App „FC Koeppchen“ (`fussball.html` im Repo `strakekine/panini-wm-2026`) — **die gehört
nicht hierher und wird von hier aus nie angefasst** (vom Nutzer, ausdrücklich). Dort steht in
`CLAUDE.md` die ganze Vorgeschichte (FLF-Schnittstelle, FuPa, Liveticker, BGL-Namen, Kalender,
iPhone-Regeln); was hier gilt, steht in `README.md`.

Kurz, was man wissen muss:
- **Ein Repo, ein Workflow, ein Cloudflare-Worker für alle Vereine** (vom Nutzer: ein Repo für alle;
  Worker-Frage Claude überlassen → einer für alle, er liest `vereine.json`). Neuer Verein:
  `tools/suche.mjs`, dann `tools/neu.mjs verein.json logo.png --abruf`. Logo und zwei Farben kommen
  vom Nutzer (FuPa hat Logos, `image.fupa.net` ist in der Cloud-Umgebung gesperrt).
- `index.html` je Verein ist **erzeugt** — Änderungen gehören in `vorlage/index.html` bzw.
  `tools/neu.mjs`, danach `node tools/neu.mjs --alle` (zählt `FASSUNG` hoch; am iPhone erst die
  Fassung prüfen lassen, Zwischenspeicher). Der Test schlägt an, wenn das vergessen wurde.
- Liveticker (Seniors 1, Nationalteam) **nur über den Worker**, kein GitHub-Rückfall wie bei
  Koeppchen. Leer, wenn bei FuPa niemand tickert.
  Worker eingerichtet vom Nutzer am 19.10.2026: `https://fussball-vereine.rjtp6r9s9h.workers.dev/`
  (eigener Worker neben dem Koeppchen-Worker, den nicht anfassen). Domain in der Cloud-Umgebung
  freigegeben. Am 19.10. geprüft: Kehlen–Yellow Boys 5:0 per `verein`+`anstoss` gefunden,
  Länderspiel per Slug, fremder Slug/unbekannter Verein/zu alter Anstoß → 400, CORS `*`.
  Ob der Takt (`scheduled`, Secret `GITHUB_TOKEN`) anstößt, zeigen die Läufe auf GitHub.
- **Aus Koeppchen nachgezogen bis `9081a07`** (20.10.2026, Koeppchen-Fassung k): Countdown, ehrlicher
  Ticker, 📍 überall, Forfait im Voraus, BGL-Datum, Unsere Woche, Abrufzeiten 7:00/22:15/23:30.
  Nicht übernommen (vom Nutzer): Ticker Seniors 2/3, anonymer Zähler. Nächster Abgleich: Basis ist
  `fussball.html` bei `9081a07`, `git merge-file` gegen `vorlage/index.html`, dann Koeppchen-Eigenes
  (Namen, `fc-koeppchen`, Reserven, Zähler) wieder herausnehmen. Eigene Ergänzungen der Vorlage:
  `fupaGedreht`/`nachFlf` (Heim/Gast bei FuPa andersherum, auch im LIVE-Kasten), `&verein=` bei jeder
  Worker-Frage, Woche-Knopf bricht bei 320 px um statt abzuschneiden.
- **Worker-Code geändert → beim Nutzer neu einfügen lassen** (Cloudflare, Edit code). Zuletzt
  20.10.2026 (Abrufzeiten).
- `slug` eines Vereins nie ändern (Kalender-UIDs, Speicherschlüssel `fussball_<slug>`; alle Apps
  teilen sich die Herkunft `strakekine.github.io`).
- Unten „Gemacht von Laurent R.“ **nur als Name, ohne Logo**: das Koeppchen-Logo hat in fremden
  Vereins-Apps nichts zu suchen (keine Erlaubnis des FC Koeppchen; vom Nutzer, 19.10.2026).
- iPhone-Regeln wie in panini-wm-2026: kein `viewport-fit=cover`, Home-Icon PNG, `navigator.share`
  nur mit `text`.
- Testen: `node test/fussball.mjs` (läuft auch vor jedem Abruf). Abruf live in der Cloud:
  `NODE_USE_ENV_PROXY=1 node tools/fussball.mjs [ordner …]`.
- Die Action committet auf `main` — vor dem Push `git pull --rebase`.
