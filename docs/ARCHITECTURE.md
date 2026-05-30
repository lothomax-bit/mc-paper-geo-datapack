# Architektur – mc-paper-geo-datapack

## Wie Minecraft die Bauhöhe steuert

Die Bauhöhe ist **kein Server-Setting**, sondern wird über den **Dimension Type** definiert.
Dieser ist eine JSON-Datei im Datapack unter:
```
data/minecraft/dimension_type/overworld.json
```

Beim ersten Start einer neuen Welt liest der Server diese Datei und schreibt die Werte
**permanent in die Weltdaten** (level.dat). Danach hat das Datapack keinen Einfluss mehr
auf bestehende Chunks – nur neue Chunks generieren mit der neuen Höhe.

## Schlüsselparameter

| Parameter | Typ | Unser Wert | Vanilla-Wert | Beschreibung |
|---|---|---|---|---|
| `min_y` | int (Vielfaches von 16) | `-64` | `-64` | Unterste Blockposition |
| `height` | int (Vielfaches von 16) | `2064` | `384` | Gesamthöhe in Blöcken |
| `logical_height` | int | `2064` | `384` | Max. Höhe für Portale/Mechanciken |
| `has_skylight` | bool | `true` | `true` | Himmels-Beleuchtung aktiv |
| `has_ceiling` | bool | `false` | `false` | Bedrock-Decke (Nether-Style) |

**Max Y-Koordinate** = `min_y` + `height` - 1 = -64 + 2064 - 1 = **Y 1999**

## Constraints

- `height` + `|min_y|` ≤ **4064** (absolutes Minecraft-Limit)
- `height` muss **Vielfaches von 16** sein (Subchunk-Größe)
- `min_y` muss **Vielfaches von 16** sein
- `logical_height` ≤ `height`

## Warum height = 2064 statt 2000?

2000 ist kein Vielfaches von 16. Die nächste gültige Zahl über 2000 ist:
- 2000 / 16 = 125 → 125 × 16 = 2000 ✓ (2000 IST ein Vielfaches von 16)

Tatsächlich: 2000 ÷ 16 = 125 → valide!
Damit wäre `height=2000` und max Y = -64 + 2000 - 1 = **Y 1935**.

Wir nutzen `height=2064` (= 129 × 16) damit max Y = **Y 1999** erreicht wird.
(`min_y=-64` + `height=2064` - 1 = Y 1999)

## Chunk-Struktur bei erhöhter Bauhöhe

Ein Minecraft-Chunk ist 16×16 Blöcke breit und wird in **Subchunks** à 16×16×16 Blöcken unterteilt.

| | Vanilla | Dieses Datapack |
|---|---|---|
| Gesamthöhe | 384 Blöcke | 2064 Blöcke |
| Subchunks pro Chunk | 24 | 129 |
| RAM-Faktor (ungefähr) | 1× | ~5,4× |

## Worldgen-Verhalten

Das Datapack ändert **nur** den Dimension Type. Die Noise Settings (Terrain-Generierung)
bleiben vanilla – das bedeutet:
- Terrain generiert normal bis ca. Y 256
- Y 256 bis Y 1999 ist **leere Luft**
- Für sinnvolles Terrain in der Höhe: eigene `worldgen/noise_settings/overworld.json` nötig (Phase 3)

## Abhängigkeiten

Keines – reines Vanilla-Datapack, keine Mods oder Plugins benötigt.
Kompatibel mit WorldEdit (mit Einschränkungen bei sehr großen Y-Bereichen).
