# Quellen fern abschalten (sources.json) – ab App v3.4

Die App lädt `sources.json` aus diesem Repo (neben `articles.json`, höchstens alle 3 Std.).
Damit lässt sich eine Preisquelle **ohne App-Update abschalten**, z. B. wenn ein Shop seine Seite umbaut
oder Abfragen nicht mehr erlaubt.

- `"enabled": false` → Quelle wird in Suche, Budget-Planer und Hintergrund-Wächter übersprungen; die App zeigt `note` als Grund.
- `"enabled": true` oder fehlender Eintrag → wie vom Nutzer in den Optionen eingestellt (die Datei kann nur **abschalten**, nie heimlich einschalten).
- Datei nicht erreichbar/ungültig → letzte gültige Version bzw. alles wie eingestellt.
- Schlüssel = interne Namen: GEIZHALS, ALTERNATE, IDEALO, BILLIGER, NBB, PROSHOP, AMAZON_DE/FR/IT/ES, EBAY, BACKMARKET, FEED.

Beispiel:
```json
"PROSHOP": { "enabled": false, "note": "Shop-Seite umgebaut – Fix folgt" }
```
Danach: `updated` setzen, `python3 -m json.tool sources.json`, commit + push (kein force-push).
