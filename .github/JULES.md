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
├── pack.mcmeta                        ← IMMER bearbeiten wenn pack_format sich ändert
├── data/
│   └── minecraft/
│       └── dimension_type/
│           ├── overworld.json         ← KERN-DATEI: Bauhöhe-Definition
│           └── overworld_caves.json   ← Caves-Variante, muss konsistent mit overworld.json sein
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

## Referenz-Links

- Minecraft Wiki Dimension Type: https://minecraft.wiki/w/Dimension_type
- Misode Dimension Generator: https://misode.github.io/dimension/
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
fix(overworld): correct height to be multiple of 16
docs(paper): add chunk loading workaround for Paper 1.21.x
chore: update pack_format to 101 for MC 26.1.2
```
