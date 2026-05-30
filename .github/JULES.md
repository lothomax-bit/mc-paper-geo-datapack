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
│           ├── density_function/            ← NEU: Inline Density Functions
│           │   └── overworld/
│           │       ├── depth.json
│           │       └── ridges_folded.json
│           └── noise_settings/
│               └── overworld.json           ← Terrain-Generierung
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

### Task-Typ 4: Noise Settings (Terrain-Generierung) – Referenz-Task (veraltet)

Siehe Task-Typ 5 – die reine `noise_settings/overworld.json` Anpassung reicht NICHT aus.

### Task-Typ 5: Terrain-Höhe KORREKT skalieren – Density Functions

#### Warum Task 4 allein nicht funktioniert (Root Cause)

Das Problem: Die aktuelle `noise_settings/overworld.json` referenziert Vanilla Density Functions
über Pfad-Strings wie `"minecraft:overworld/depth"` und `"minecraft:overworld/ridges"`.
Diese Vanilla-internen Density Functions sind **hardcodiert** und kennen nur den Vanilla-Höhenbereich
bis ca. Y 256–320. Das `noise` Block mit `height: 2064` definiert nur den **verfügbaren Raum**,
nicht die tatsächlich generierten Berg-Höhen.

**Der Terrain-Generator berechnet die Blockdichte anhand der `depth`-Density-Function.**
Diese liefert bei Y > ~320 bereits negative Werte (= Luft) – egal wie groß `noise.height` ist.

#### Lösung: Eigene Density Functions

Die Vanilla-Referenzen müssen durch **eigene, inline definierte oder datapack-lokale**
Density Functions ersetzt werden, die den neuen Höhenbereich kennen.

**Ansatz A – Inline in noise_settings (einfacher):**
Die `noise_router`-Felder `depth`, `ridges` etc. direkt als verschachteltes JSON-Objekt
angeben statt als String-Referenz. So werden Vanilla-Density-Functions vollständig überschrieben.

**Ansatz B – Eigene density_function JSON-Dateien (sauberer):**
Dateien unter `data/minecraft/worldgen/density_function/overworld/depth.json` etc. anlegen
und in `noise_settings/overworld.json` als `"minecraft:overworld/depth"` referenzieren
(Datapack-Dateien überschreiben Vanilla-Dateien unter gleichem Namespace+Pfad).

**Empfehlung: Ansatz B verwenden.**

#### Konkrete Implementierung (Ansatz B)

**Schritt 1: Vanilla Density Functions als Basis laden**
Vanilla-Originale unter:
- `depth`: https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/density_function/overworld/depth.json
- `ridges`: https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/density_function/overworld/ridges.json
- `ridges_folded`: https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/density_function/overworld/ridges_folded.json
- `sloped_cheese`: https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/density_function/overworld/sloped_cheese.json
- `continents`: https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/density_function/overworld/continents.json
- `erosion`: https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/density_function/overworld/erosion.json

**Schritt 2: Skalierungsfaktor anwenden**
Der Skalierungsfaktor für Y 1700 beträgt **~6.6** (1700 / 256 ≈ 6.64).

In `depth.json` muss der `y_scaled`-Operator (oder äquivalente Noise-Berechnung)
so angepasst werden, dass die Density-Kurve um Faktor 6.6 gestreckt wird.

Konkret: In Vanilla berechnet `depth` einen Wert der bei Y ≈ 256 auf 0 fällt (= Luft).
Nach der Skalierung soll dieser 0-Übergang erst bei Y ≈ 1700 stattfinden.

Beispiel für die Skalierung über einen `mul`-Knoten:
```json
{
  "type": "minecraft:mul",
  "argument1": 6.6,
  "argument2": {
    "type": "minecraft:y_clamped_gradient",
    "from_y": -64,
    "to_y": 1700,
    "from_value": 1.5,
    "to_value": -1.5
  }
}
```

**Schritt 3: sloped_cheese skalieren**
`sloped_cheese.json` kontrolliert die Bergspitzen. Die `offset`-Werte müssen
proportional zum Skalierungsfaktor angepasst werden.

**Schritt 4: Caves begrenzen**
In `noise_settings/overworld.json` gibt es einen `large_ore_veins_enabled` und
`aquifers_enabled` Block sowie Cave-spezifische Density Functions
(`minecraft:overworld/caves/...`). Diese Cave-Density-Functions ebenfalls
als eigene Dateien unter `data/minecraft/worldgen/density_function/overworld/caves/`
überschreiben und den Cave-Erzeugungsbereich auf max. Y 900 begrenzen.

#### Höhen-Übersicht (Zielwerte):

| Bereich | Y-Koordinate | Beschreibung |
|---|---|---|
| Tiefstes Bedrock | -64 | Vanilla-Standard |
| Meeresspiegel | 63 | Vanilla-Standard, nicht ändern |
| Normale Oberfläche | 64–256 | Vanilla-Terrain-Bereich |
| Bergketten (max.) | bis **Y 1700** | Ziel dieser Anpassung |
| Luft-Puffer | Y 1700–1999 | Leer, Spielraum für Bauten |
| Max. Bauhöhe | Y 1999 | Durch dimension_type definiert |
| Höhlen (max.) | bis **Y 900** | Skalierte Cave-Generierung |

#### Validierung:
1. JSON-Syntax aller neuen Dateien prüfen
2. Im Spiel: Mit `/tp @s ~ 1500 ~` testen ob dort Terrain (Stein) existiert
3. Mit `/tp @s ~ 1750 ~` testen ob dort Luft ist
4. Alle Density Function Dateien unter: https://misode.github.io/worldgen/density-function/ validieren
5. `sea_level` in noise_settings muss `63` bleiben

#### Commit nach Fertigstellung:
```
feat(worldgen): replace vanilla density functions to scale terrain to Y 1700
```

## Referenz-Links

- Minecraft Wiki Density Functions: https://minecraft.wiki/w/Density_function
- Minecraft Wiki Noise Settings: https://minecraft.wiki/w/Custom_world_generation
- Misode Density Function Generator: https://misode.github.io/worldgen/density-function/
- Misode Noise Settings Generator: https://misode.github.io/worldgen/noise-settings/
- Vanilla density_function Ordner: https://github.com/misode/mcmeta/tree/data/data/minecraft/worldgen/density_function/overworld
- Vanilla noise_settings/overworld.json: https://github.com/misode/mcmeta/blob/data/data/minecraft/worldgen/noise_settings/overworld.json
- Minecraft Wiki Dimension Type: https://minecraft.wiki/w/Dimension_type
- Misode Dimension Generator: https://misode.github.io/dimension/
- Pack Format Liste: https://minecraft.wiki/w/Pack_format
- PaperMC Docs: https://docs.papermc.io/paper/reference/world-configuration/
- Version Explorer: https://misode.github.io/versions/

## Commit-Konventionen

Format: `type(scope): beschreibung`

Types:
- `feat`: Neue Funktion
- `fix`: Bugfix
- `docs`: Nur Dokumentation
- `chore`: Metadaten, pack_format-Updates
- `test`: Test-Welten, Validierungsskripte

Beispiele:
```
feat(dimension): raise overworld height to 2064 blocks
feat(worldgen): replace vanilla density functions to scale terrain to Y 1700
fix(overworld): correct height to be multiple of 16
docs(paper): add chunk loading workaround for Paper 1.21.x
chore: update pack_format to 101 for MC 26.1.2
```
