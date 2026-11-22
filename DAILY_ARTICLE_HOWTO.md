# Täglich einen Ratgeber-Artikel anfügen

Repo: `pmzeus82-dev/pc-preisfinder-ratgeber`, Branch `main`, Datei `articles.json` (Repo-Wurzel).
App-URL: `https://raw.githubusercontent.com/pmzeus82-dev/pc-preisfinder-ratgeber/main/articles.json`

## Was die App erwartet
- Gültiges JSON, UTF-8, **max. 2 MB** (größere Feeds werden verworfen → App zeigt Cache bzw. mitgelieferte Artikel). Ab ca. 1,5 MB die ältesten Artikel entfernen (die 10 Startartikel sind ohnehin in der App eingebaut).
- Struktur: `{ "version": 1, "title": "...", "updated": "YYYY-MM-DD", "articles": [ ... ] }`
- Die App sortiert selbst nach `date` (neueste zuerst). Konvention trotzdem: **neuen Artikel oben** in `articles` einfügen.
- Ein Artikel mit gleicher `id` wie ein vorhandener (auch eingebauter) **ersetzt** diesen → für Korrekturen dieselbe `id` behalten, für neue Artikel immer eine neue.
- „Neu“-Badge und die optionale Benachrichtigung hängen an der `id`: niemals die `id` eines veröffentlichten Artikels ändern (sonst erscheint er erneut als neu).
- Die App lädt beim Start höchstens alle 3 Std. und im Hintergrund alle 12 Std. → ein Artikel pro Tag ist passend. raw.githubusercontent.com cached ca. 5 Min.

## Artikel-Felder
| Feld | Pflicht | Regeln |
|---|---|---|
| `id` | ja | Kleinbuchstaben, Ziffern, Bindestriche, eindeutig, sprechend, z. B. `ssd-pcie4-vs-pcie5` |
| `date` | ja | `YYYY-MM-DD` (Europe/Berlin, Veröffentlichungstag), nicht in der Zukunft |
| `title` | ja | max. ~70 Zeichen |
| `teaser` | ja | 1–2 Sätze für die Liste |
| `body` | ja | Markdown-Teilmenge: `#`/`##`/`###`, Listen `-`/`1.`, `**fett**`, `*kursiv*`, Links `[Text](https://…)`. Kein HTML, keine Bilder im Text (Titelbild nur über `image`). Zeilenumbrüche als `\n` im JSON |
| `summary` | **ja (ab v3.2)** | „In 30 Sekunden“-Kasten: 2–3 sachliche Sätze, max. 600 Zeichen, nur Fakten aus dem Artikel |
| `tags` | empfohlen | z. B. `["Netzteil","PC-Bau"]` – erster Tag erscheint in der Liste |
| `image` | empfohlen | Titelbild, nur `https://raw.githubusercontent.com/pmzeus82-dev/pc-preisfinder-ratgeber/main/img/<id>.webp` (siehe „Titelbild“) – oder Feld weglassen |
| `imageCredit` | ja, wenn `image` | Urheber laut Commons, z. B. `"Jacek Halicki / Wikimedia Commons"` |
| `imageLicense` | ja, wenn `image` | exakt der Commons-Kurzname, z. B. `"CC BY-SA 4.0"`, `"CC0"`, `"Public domain"` |
| `imageSource` | ja, wenn `image` | `https://commons.wikimedia.org/wiki/File:…` (Dateiseite) |
| `author` | nein | z. B. `"Redaktion"` |
| `links` | empfohlen | Sprünge in die App (siehe unten) |

`links[]`: `{ "label": "Netzteile ab 700 W, 80 PLUS Gold", "category": "NETZTEIL", "sub": null, "filters": {"psu_watt":["ab 700 W"],"psu_eff":["ab Gold"]}, "advisor": false, "screen": null }`
- `category`: Enum-Name, z. B. CPU, MAINBOARD, RAM, GPU, SSD, NETZTEIL, GEHAEUSE, KUEHLER, LUEFTER, MONITOR, MAUS, TASTATUR, AUDIO, WEBCAM, CARDREADER, HUB_DOCK, CONTROLLER, FLIGHTSIM, SIMRACING, KOMPLETT_PC, NOTEBOOK, DRUCKER, EXT_SPEICHER, NETZWERK
- `sub`: Unterkategorie-Label wie in der App (z. B. „OLED“, „NAS“), sonst weglassen
- `filters`: Gruppen-ID → Options-Labels **exakt** wie im Quellcode (`model/Filters.kt`, `model/ExtraFilters.kt`); Unbekanntes wird ignoriert
- `advisor: true` öffnet den Kaufberater der Kategorie; `screen`: `"merkliste"` oder `"filter"`

## Titelbild (lizenzgeprüft – sonst kein Bild)
1. Passendes **echtes Foto** auf Wikimedia Commons suchen: `python3 /workspace/pc-preis-app/tools/commons_search.py "suchbegriff"` (zeigt Lizenz, Urheber, `ok`).
2. Nur verwenden, wenn `ok=True`, d. h. Lizenz **Public domain / CC0 / CC BY x.0 / CC BY-SA x.0**. Nicht verwenden: NC, ND, „Fair use“, unklare Lizenz, Logos/Marken als Hauptmotiv, erkennbare Personen, Querformat < 1000 px. Im Zweifel: **kein Bild** (Felder `image*` weglassen) – die App zeigt dann einfach keinen Header.
3. Auf 1200×630 zuschneiden und als WebP speichern: `img/<id>.webp` (Ziel < 200 KB). Beispiel:
   ```bash
   python3 - <<'PY'
   import sys; sys.path.insert(0, "/workspace/pc-preis-app/tools")
   from fetch_tile_images import fetch
   c = fetch("Dateiname auf Commons.jpg", "img/<id>.webp", (1200, 630), quality=76)
   print(c["author"], c["license"], c["page"])
   PY
   ```
   (Bei HTTP 429 von Wikimedia: einige Minuten warten bzw. die Datei mit `curl -A "PC-Preisfinder (pmzeus82-dev)"` laden und mit Pillow zuschneiden.)
4. `imageCredit` = `"<author> / Wikimedia Commons"`, `imageLicense` = Lizenz, `imageSource` = Commons-Seite. Die App zeigt „Foto: … · Lizenz · Quelle“ unter dem Bild – das ist die Namensnennung, ohne sie kein Bild.
5. Eintrag in `img/CREDITS.json` ergänzen (title, page, author, license, licenseUrl, file, modified="zugeschnitten und verkleinert").

## Inhaltliche Regeln
Faktisch, deutsch, verständlich. **Keine erfundenen Testergebnisse, Benchmarks oder Preise**; keine Kaufempfehlung für ein konkretes Produkt als „Testsieger“. Herstellerangaben als solche kennzeichnen.

## Ablauf (git-Klon auf der Box, gh als pmzeus82-dev angemeldet)
Arbeitskopie: `/workspace/ratgeber-repo` (Branch `main`). Zugangsdaten über `gh auth setup-git` (einmalig; bei „Authentication failed“ `gh auth status` prüfen).

```bash
cd /workspace/ratgeber-repo
git checkout main && git pull --ff-only            # 1. aktuellen Stand holen

# 2. Artikel oben einfügen + "updated" setzen (neuer Artikel in /tmp/new_article.json)
python3 - <<'PY'
import json, datetime
feed = json.load(open("articles.json", encoding="utf-8"))
new  = json.load(open("/tmp/new_article.json", encoding="utf-8"))
assert all(new.get(k) for k in ("id", "date", "title", "teaser", "body", "summary")), "Pflichtfeld fehlt"
if new.get("image"):
    import os; assert all(new.get(k) for k in ("imageCredit", "imageLicense", "imageSource")), "Bildnachweis fehlt"
    assert os.path.exists("img/" + new["image"].rsplit("/", 1)[1]), "Bilddatei fehlt in img/"
assert new["id"] not in {a["id"] for a in feed["articles"]}, "id schon vergeben"
feed["articles"].insert(0, new)
feed["updated"] = datetime.date.today().isoformat()
open("articles.json", "w", encoding="utf-8").write(json.dumps(feed, ensure_ascii=False, indent=2) + "\n")
PY

# 3. validieren (gültiges JSON, Größe < 1,5 MB)
python3 -c "import json,os;json.load(open('articles.json'));assert os.path.getsize('articles.json')<1_500_000;print('ok')"

# 4. committen und pushen
git add articles.json img/
git commit -m "Ratgeber: <Titel> (YYYY-MM-DD)"
git push origin main || { git pull --rebase origin main && git push origin main; }   # 5. bei Ablehnung: rebase + erneut
```
- Bei einem Rebase-Konflikt in `articles.json`: `git rebase --abort`, `git reset --hard origin/main`, Schritte 2–4 erneut ausführen (nie `--force` pushen).
- **Prüfen:** `curl -s https://raw.githubusercontent.com/pmzeus82-dev/pc-preisfinder-ratgeber/main/articles.json | python3 -m json.tool >/dev/null && echo OK` (raw-CDN ggf. ~5 Min. verzögert).

## Vorlage
```json
{
  "id": "beispiel-artikel-id",
  "date": "2026-11-21",
  "title": "Titel",
  "teaser": "Kurzbeschreibung.",
  "summary": "Das Wichtigste in 2–3 Sätzen.",
  "image": "https://raw.githubusercontent.com/pmzeus82-dev/pc-preisfinder-ratgeber/main/img/beispiel-artikel-id.webp",
  "imageCredit": "Urheber / Wikimedia Commons",
  "imageLicense": "CC BY-SA 4.0",
  "imageSource": "https://commons.wikimedia.org/wiki/File:Beispiel.jpg",
  "body": "## Abschnitt\nText mit **Hervorhebung**.\n\n- Punkt 1\n- Punkt 2",
  "tags": ["Kategorie"],
  "links": [ { "label": "Passende Filter", "category": "SSD", "filters": {}, "advisor": true } ]
}
```
