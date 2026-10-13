# GPS Chase – Kartendaten

Kartenpakete für die App „GPS Chase“ (iPhone und Apple Watch): benannte Straßen und Adressen aus OpenStreetMap für Deutschland, Österreich und die Schweiz, aufgeteilt in Kacheln von rund 3 × 3 km.

Die Pakete liegen unter [Releases](../../releases). Die App lädt daraus per Teilabruf nur die Kacheln ihrer Umgebung.

*Map packs for the app “GPS Chase”: named roads and addresses from OpenStreetMap for Germany, Austria and Switzerland, split into tiles of about 3 × 3 km. See [Releases](../../releases).*

## Lizenz / License

Die Daten stammen von [OpenStreetMap](https://www.openstreetmap.org/copyright) und stehen unter der [Open Database License (ODbL) 1.0](https://opendatacommons.org/licenses/odbl/1-0/).

© OpenStreetMap-Mitwirkende / OpenStreetMap contributors. Auszüge / extracts: [Geofabrik](https://download.geofabrik.de).

## Inhalt eines Pakets

Jede Veröffentlichung enthält zwei Dateien:

| Datei | Inhalt |
|---|---|
| `manifest.json` | Datenstand, Name und Größe der Paketdatei und je Region der Bereich in der Paketdatei |
| `dach-<Datenstand>.gpca` | alle Kacheln in einer Datei |

Enthalten sind Wege mit `highway=*` und Namen (dazu Kreisverkehre) sowie Objekte mit `addr:street` und `addr:housenumber`. Als abgedeckt gelten nur Kacheln, die vollständig innerhalb der Grenze des Auszugs liegen.

### Raster

Eine Kachel misst 0,03° Breite × 0,045° Länge. Zeile `cy = floor((Breite + 90) / 0,03)`, Spalte `cx = floor((Länge + 180) / 0,045)`, Schlüssel `cy × 8000 + cx`. Eine Region bündelt 16 × 16 Kacheln und heißt `<cy div 16>_<cx div 16>`.

### manifest.json

```json
{
  "format": 1,
  "version": "2026-10-12",
  "file": "dach-2026-10-12.gpca",
  "base": "https://…/releases/download/2026-10-12/",
  "size": 388300000,
  "compression": "deflate",
  "regions": { "293_259": [8, 6400000] }
}
```

`regions` nennt je Region `[Anfang, Länge]` in Bytes innerhalb der Paketdatei.

### Paketdatei (.gpca)

- Kopf, 8 Bytes: `GPCA`, Version (1), drei Bytes reserviert
- je Region ein Block:
  - Verzeichnis, 2048 Bytes: 256 Einträge `[Anfang u32, Länge u32]` (little-endian), Platz `(cy mod 16) × 16 + (cx mod 16)`. `Anfang` zählt ab dem Ende des Verzeichnisses. `Anfang = 0xFFFFFFFF` heißt „nicht abgedeckt“, `Länge = 0` heißt „abgedeckt, aber leer“.
  - danach die Kacheln, jede einzeln mit Deflate (ohne zlib-Kopf) gepackt

### Kachel (entpackt)

Zahlen ohne feste Breite sind LEB128-Varints, Texte sind Länge + UTF-8, Koordinaten stehen in 10⁻⁷ Grad und sind zickzack-kodierte Differenzen: der erste Punkt relativ zur Südwest-Ecke der Kachel, jeder weitere relativ zum Vorgänger.

1. `GPC1` (4 Bytes), Kachelschlüssel (u32, little-endian)
2. Namen: Anzahl, dann je Name ein Text
3. Wege: Anzahl, dann je Weg: OSM-Nummer, Namensverweis (0 = ohne, sonst Platz in der Namensliste ab 1), Klasse (1 Byte), Merkmale (1 Byte), Punktanzahl, je Punkt Breite und Länge
4. Adressen: Anzahl der Straßen, dann je Straße: Namensverweis, Anzahl, je Adresse Hausnummer (Text), Breite und Länge

Klassen: 0 motorway, 1 trunk, 2 primary, 3 secondary, 4 tertiary, 5 unclassified, 6 residential, 7 living_street, 8 pedestrian, 9 service, 10 track, 11 footway, 12 path, 13 cycleway, 14 steps, 15 sonstige. Merkmale: 1 Kreisverkehr, 2 Fläche (`area=yes`), 4 Verbindungsrampe (`*_link`).
