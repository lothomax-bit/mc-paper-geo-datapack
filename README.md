# mc-paper-geo-datapack

Minecraft Datapack für **Paper-Server** (Minecraft 26.1.2) zur Anhebung der maximalen Bauhöhe auf **2000 Blöcke** (Y -64 bis Y 1999).

## Ziel

- Bauhöhe von Standard 384 Blöcken auf **2064 Blöcke gesamt** anheben
- Kompatibel mit Paper 1.21.x / 26.1.2
- Terrain-Generierung bis zur neuen Höhe sinnvoll angepasst
- Performance-Empfehlungen für Paper inbegriffen

## Schnellstart

1. Datapack als `.zip` packen oder Ordner direkt in `world/datapacks/` legen
2. **Neue Welt starten** (bestehende Welten können nicht nachträglich erweitert werden)
3. Paper-Server starten und mit `/datapack list` prüfen
4. Mit `/execute in minecraft:overworld run tp @s ~ 1500 ~` die neue Höhe testen

## Projektstruktur

```
mc-paper-geo-datapack/
├── pack.mcmeta                              ← Datapack-Metadaten (pack_format 101)
├── data/
│   └── minecraft/
│       └── dimension_type/
│           ├── overworld.json               ← Bauhöhe 2064, min_y -64
│           └── overworld_caves.json         ← Caves-Variante angepasst
├── docs/
│   ├── ARCHITECTURE.md                      ← Technische Architektur
│   ├── PAPER_COMPATIBILITY.md               ← Paper-spezifische Hinweise
│   └── PERFORMANCE.md                       ← Performance-Tuning
└── .github/
    └── JULES.md                             ← Kontext für Google Jules
```

## Technische Details

| Parameter | Wert |
|---|---|
| Minecraft Version | 26.1.2 |
| pack_format | 101 |
| `min_y` | -64 |
| `height` | 2064 |
| Max Bauhöhe (Y) | 1999 |
| Paper Version | 1.21.x / 26.1.x |

## Wichtige Hinweise

- ⚠️ Nur für **neue Welten** – bestehende Welten können NICHT nachträglich verändert werden
- ⚠️ Hoher RAM-Bedarf durch größere Chunks (~5× mehr Subchunks)
- ⚠️ Paper hat bekannte Einschränkungen bei Höhen >319 – siehe `docs/PAPER_COMPATIBILITY.md`

## Lizenz

MIT
