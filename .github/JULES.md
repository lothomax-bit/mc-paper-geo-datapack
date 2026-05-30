# Jules Agent Context – mc-paper-geo-datapack

Diese Datei ist der primäre Kontext für den Google Jules AI-Agenten.
Jules soll eigenständig Aufgaben in diesem Repository ausführen können.

## Projektbeschreibung

Ein Minecraft **Data Pack** für Paper-Server (Version 26.1.2), das die maximale Bauhöhe
des Overworld-Dimensions-Typs auf **2064 Blöcke** anhebt (Y -64 bis Y 1999).

## Stack & Technologien

- **Format**: Minecraft Data Pack (JSON-Dateien, keine Kompilierung nötig)
- **Minecraft Version**: 26.1.2 (Java Edition)
- **pack_format**: 101 (für Minecraft 26.1.x)
- **Server**: PaperMC (paper-1.21.x)
- **Sprache**: JSON ausschließlich (keine Skripte)
- **Validierung**: `misode.github.io` Generatoren + Minecraft Wiki

## Verzeichnisstruktur

```
mc-paper-geo-datapack/
├── pack.mcmeta                              ← IMMER bearbeiten wenn pack_format sich ändert
├── data/
│   └── minecraft/
│       ├── dimension_type/
│       │   ├── overworld.json               ← KERN-DATEI: Bauhöhe-Definition
│       │   └── overworld_caves.json         ← Caves-Variante, muss konsistent sein
│       └── worldgen/
│           └── noise_settings/
│               └── overworld.json           ← Terrain-Generierung (Task 4)
└── docs/
    ├── ARCHITECTURE.md
    ├── PAPER_COMPATIBILITY.md
    └── PERFORMANCE.md
```

## Wichtige Constraints für Jules

### JSON-Validierungsregeln
- `height` in `dimension_type/*.json` **muss ein Vielfaches von 16** sein
- `height` + `min_y` darf **4064 nicht überschreiten** (Minecraft Hard-Limit)
- `logical_height` darf `height` nicht überschreiten
- `min_y` muss durch **16 teilbar** sein

### Paper-Kompatibilität
- Paper hat bekannte Chunk-Lade-Probleme bei Y > 319 in älteren Versionen
- Immer `docs/PAPER_COMPATIBILITY.md` prüfen bevor Änderungen an Dimension-Typen gemacht werden
- Keine `spigot.yml` oder `paper-world-defaults.yml` Dateien in dieses Repo committen
  (diese gehören zum Server, nicht zum Datapack)

### Nicht erlaubte Änderungen
- `min_y` darf nicht unter `-2032` sinken
- `has_skylight` darf für den Overworld nicht auf `false` gesetzt werden
- Vanilla-Biome-IDs dürfen nicht umbenannt werden

## Häufige Aufgaben für Jules

### Task-Typ 1: pack_format aktualisieren
Wenn die Minecraft-Version sich ändert:
1. `pack.mcmeta` → `pack_format` auf neuen Wert setzen
2. `pack.mcmeta` → `supported_formats` Array aktualisieren
3. `README.md` → Versionstabelle aktualisieren
4. Aktuellen pack_format unter: https://cmdgen.eufonia.studio/versions/ nachschlagen

### Task-Typ 2: Bauhöhe ändern
Wenn `height` geändert wird:
1. `data/minecraft/dimension_type/overworld.json` → `height` + `logical_height` ändern
2. `data/minecraft/dimension_type/overworld_caves.json` → gleiche Werte setzen
3. `README.md` → Tabelle mit neuen Werten aktualisieren
4. Sicherstellen: `height` ist Vielfaches von 16, `height` + `min_y` ≤ 4064

### Task-Typ 3: Dokumentation aktualisieren
- Immer `docs/ARCHITECTURE.md` aktuell halten
- Bei Paper-Bugs: `docs/PAPER_COMPATIBILITY.md` ergänzen
- Quellen immer mit Datum und Minecraft-Version verlinken

### Task-Typ 4: Noise Settings (Terrain-Generierung) erstellen/anpassen

Diese Aufgabe betrifft `data/minecraft/worldgen/noise_settings/overworld.json`.

#### Designentscheidungen (verbindlich, nicht zur Diskussion):

**1. Art der Generierung: Gestreckte/skalierte Berge**
- Die bestehende Vanilla-Terrain-Logik (Extreme Hills / Windswept Hills etc.) soll
  **vertikal gestreckt** werden, sodass Berge organisch bis Y 800 ragen können.
- KEIN flaches Terrain mit schwebenden Inseln darüber.
- KEIN abrupter Bruch bei Y 256 – das Terrain soll nahtlos und natürlich wirken.
- Ziel: Wer die Welt von unten nach oben fliegt, soll ein kontinuierlich ansteigendes
  Gebirge erleben, keine künstliche Grenze.

**2. Sea Level (Meeresspiegel)**
- Bleibt bei **Y 63** (Vanilla-Standard).
- Nicht verändern.

**3. Dateipfad**
- Korrekt: `data/minecraft/worldgen/noise_settings/overworld.json`
- Diese Datei **überschreibt** die Vanilla Noise Settings für den Overworld.

**4. Basis-Datei**
- Ja: Vanilla `overworld.json` für Minecraft 1.21 als Basis verwenden.
- Referenz (Vanilla-Original): https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/noise_settings/overworld.json
- Nur die relevanten Skalierungs-Parameter anpassen, alles andere vanilla-kompatibel lassen.

**5. Höhlen (Caves)**
- Höhlen sollen **ebenfalls nach oben skaliert** werden, passend zur Terrain-Streckung.
- Konkret: Höhlen können bis ca. Y 400–500 vorkommen (entspricht der skalierten
  Vanilla-Höhlen-Tiefe relativ zur neuen Terrain-Höhe).
- Keine Höhlen über Y 500 – oberhalb soll es massives Gestein / Luft geben.

#### Konkrete Noise-Parameter-Änderungen:

Die folgenden Werte im `noise` Block der `overworld.json` anpassen:

```json
"noise": {
  "height": 2064,
  "min_y": -64,
  "size_horizontal": 1,
  "size_vertical": 2
}
```

Im `noise_router` die Terrain-Amplituden erhöhen:
- `initial_density_without_jaggedness`: Skalierungsfaktor für Bergspitzen anpassen
- `depth`: Offset so anpassen, dass Meeresboden bei Y ~-30 bleibt, Oberfläche bei Y ~100–800
- `ridges` (Peaks & Valleys): Amplitude erhöhen damit Berge bis Y 800 ragen

Die `spawn_target` Einträge so setzen:
```json
"spawn_target": [
  {"parameter": {"min": -1.0, "max": -0.97}, "value": 0.713},
  {"parameter": {"min": -0.97, "max": 0.0},  "value": 0.0},
  {"parameter": {"min": 0.0,  "max": 0.5},   "value": 0.0},
  {"parameter": {"min": 0.5,  "max": 1.0},   "value": 0.0}
]
```

#### Validierung nach der Erstellung:
1. JSON-Syntax mit einem Linter prüfen
2. Sicherstellen dass `noise.height` == `dimension_type/overworld.json` `height` (beide: 2064)
3. Sicherstellen dass `noise.min_y` == `dimension_type/overworld.json` `min_y` (beide: -64)
4. `sea_level` muss `63` sein
5. Datei unter: https://misode.github.io/worldgen/noise-settings/ validieren

#### Nach der Erstellung:
- `docs/ARCHITECTURE.md` → Abschnitt "Worldgen-Verhalten" aktualisieren
- Eintragen welche Noise-Parameter geändert wurden und warum
- Commit-Message: `feat(worldgen): scale terrain noise to Y 800 with cave extension to Y 500`

## Referenz-Links

- Minecraft Wiki Dimension Type: https://minecraft.wiki/w/Dimension_type
- Minecraft Wiki Noise Settings: https://minecraft.wiki/w/Custom_world_generation
- Misode Dimension Generator: https://misode.github.io/dimension/
- Misode Noise Settings Generator: https://misode.github.io/worldgen/noise-settings/
- Vanilla overworld.json (Referenz): https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/noise_settings/overworld.json
- Pack Format Liste: https://minecraft.wiki/w/Pack_format
- PaperMC Docs: https://docs.papermc.io/paper/reference/world-configuration/
- Version Explorer: https://misode.github.io/versions/

## Commit-Konventionen

Format: `type(scope): beschreibung`

Types:
- `feat`: Neue Funktion (z.B. neue Dimension hinzugefügt)
- `fix`: Bugfix (z.B. falsche height-Berechnung)
- `docs`: Nur Dokumentation
- `chore`: Metadaten, pack_format-Updates
- `test`: Test-Welten, Validierungsskripte

Beispiele:
```
feat(dimension): raise overworld height to 2064 blocks
feat(worldgen): scale terrain noise to Y 800 with cave extension to Y 500
fix(overworld): correct height to be multiple of 16
docs(paper): add chunk loading workaround for Paper 1.21.x
chore: update pack_format to 101 for MC 26.1.2
```
