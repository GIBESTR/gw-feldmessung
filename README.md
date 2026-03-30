# GW-Feldmessung PWA

Offline-fähige Progressive Web App für Grundwasser-Feldmessungen.
Läuft auf Android und iOS, kein App Store nötig.

---

## Dateien

| Datei | Beschreibung |
|---|---|
| `index.html` | Haupt-App |
| `sw.js` | Service Worker (Offline-Cache) |
| `manifest.json` | PWA-Manifest |
| `beispiel_projekt.json` | Beispiel-Importdatei |

---

## Deployment (einmalig)

Die App muss auf einem Webserver liegen, damit sie als PWA installiert werden kann.
**HTTPS ist Pflicht** (ausser localhost).

### Option A: Lokaler PC / Intranet
Falls die App nur im internen Netz gebraucht wird:

1. Python installiert?
   ```
   python -m http.server 8080
   ```
   → App erreichbar unter `http://[PC-IP]:8080`

2. Oder IIS / Apache / nginx auf Windows-Server

### Option B: GitHub Pages (kostenlos, HTTPS inklusive)
1. GitHub-Account erstellen
2. Neues Repository anlegen
3. Alle 4 Dateien hochladen
4. Settings → Pages → Deploy from branch: main
5. App läuft unter `https://[user].github.io/[repo]`

---

## App auf Handy installieren

### Android (Chrome)
1. App-URL im Browser öffnen
2. Menü (⋮) → „Zum Startbildschirm hinzufügen"
3. App erscheint als Icon auf dem Homescreen

### iOS (Safari – MUSS Safari sein!)
1. App-URL in Safari öffnen
2. Teilen-Symbol (□↑) → „Zum Home-Bildschirm"
3. App erscheint als Icon

---

## Import-Format (JSON)

```json
{
  "projekt": {
    "id": "eindeutige_id",
    "name": "Projektname",
    "messstellen": [
      {
        "id": "GW-001",
        "name": "Messstelle Aarberg Nord",
        "ok_rohr": 481.350,
        "ueberstand": 0.215,
        "lat": 47.0523,
        "lng": 7.2701
      }
    ]
  },
  "messungen": []
}
```

**Felder Messstellen:**
- `id` – eindeutige ID (Pflicht)
- `name` – Bezeichnung (Pflicht)
- `ok_rohr` – Oberkante Rohr in m ü.M.
- `ueberstand` – Überstand in m
- `lat` / `lng` – Koordinaten WGS84 (für Karte)

---

## Export

Export erzeugt dieselbe JSON-Struktur → direkt in PostgreSQL importierbar.
CSV-Export für Excel.

Nach dem Export werden neue Messungen als „synchronisiert" markiert.

---

## Mehrere Geräte / Sync

Jedes Gerät arbeitet offline-lokal.
Sync = Export vom Feld-Gerät → Import auf PC → In DB → Neu exportieren → Import auf anderen Geräten.

---

## PostgreSQL Import (Beispiel)

```python
import json, psycopg2

with open('GW_Export.json') as f:
    data = json.load(f)

conn = psycopg2.connect("dbname=grundwasser ...")
cur = conn.cursor()

for m in data['messungen']:
    if m['neu']:
        cur.execute("""
            INSERT INTO messungen (id, messstellen_id, datum, abstich, messbar, bemerkung)
            VALUES (%s, %s, %s, %s, %s, %s)
            ON CONFLICT (id) DO NOTHING
        """, (m['id'], m['messstellen_id'], m['datum'], m['abstich'], m['messbar'], m['bemerkung']))

conn.commit()
```
