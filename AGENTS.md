# Technischer Leitfaden & System-Check (Minecraft 26.1.2 / 1.21.2+ Datapack)

## 1. System-Check & Hard Limits

| Parameter | Wert | Hinweise |
| --- | --- | --- |
| **Projektname** | `mc-paper-geo-datapack` | Minecraft Data Pack für PaperMC |
| **Minecraft-Version** | 26.1.2 / 1.21.2+ | Java Edition |
| **Pack-Format** | 101 | Zwingende Nutzung von `pack_format` 101 (inkl. `supported_formats` falls genutzt, sowie spezifischer JSON-Strukturen) |
| **min_y** | -64 | Basis der Welt |
| **height** | 2064 | Gesamte Blockhöhe (von min_y bis Max Bauhöhe) |
| **Max Bauhöhe** | Y 1999 | min_y (-64) + height (2064) - 1 = 1999 |
| **Einschränkung** | Nur neue Welten | Hoher RAM-Bedarf; bestehende Welten können nicht erweitert werden |
| **Erweiterbarkeit** | Modular aufgebaut | Terrain/Biome sollen später einfach anpassbar/integrierbar sein |

## 2. Ordnerstruktur

Die genaue Ordnerstruktur, die für das Datapack zwingend erforderlich ist:

```text
mc-paper-geo-datapack/
├── pack.mcmeta                       # Datapack-Metadaten (Format 101)
├── README.md                         # Projektübersicht
├── AGENTS.md                         # Dieser Leitfaden
└── data/
    └── minecraft/
        ├── dimension_type/
        │   └── overworld.json        # Definition der Dimension (min_y, height, etc.)
        └── worldgen/
            ├── noise_settings/
            │   └── overworld.json    # Terrain-Generierungs-Einstellungen für die neue Höhe
            └── density_function/     # Ausgelagerte Density Functions (Zwang ab Format 101)
                └── overworld/
                    ├── ...           # Einzelne JSON-Dateien für Density Functions
```

## 3. Technische Details (Minecraft 26.1.2 / Pack-Format 101)

- **`pack.mcmeta`**: Muss `min_format` und `max_format` bzw. `supported_formats` verwenden.
- **`dimension_type`**:
  - Erfordert einen `distribution`-Typ für `monster_spawn_light_level`.
- **`noise_settings`**:
  - `spawn_target` verwendet array-basierte Parametergrenzen.
  - Müssen `surface_rule` und `noise_router.preliminary_surface_level` beinhalten, um Parsing-Fehler zu vermeiden.
- **`density_functions`**:
  - **Müssen** in eigenständige JSON-Dateien extrahiert werden, die ihre vollständigen Objektdefinitionen enthalten. Sie können nicht mehr als bloße String-Selbstreferenzen im selben JSON definiert werden.

## 4. Technical Debt / To-Do Liste

- [ ] Erstellen der `pack.mcmeta` mit `pack_format: 101` und den neuen Feldern (`min_format`, `max_format`).
- [ ] Erstellen der `data/minecraft/dimension_type/overworld.json`:
  - Anpassung der Parameter `min_y: -64` und `height: 2064`.
  - Berücksichtigung von `monster_spawn_light_level` mit Distribution-Typ.
- [ ] Erstellen der `data/minecraft/worldgen/noise_settings/overworld.json`:
  - Anhebung der Generierungsgrenzen auf 2064.
  - Anpassung der Noise-Router und Surface-Rules für Y=1999.
  - Implementierung einer modularen Struktur, um später einfache Terrain- und Biom-Anpassungen zu ermöglichen.
  - Sicherstellen, dass `noise_router.preliminary_surface_level` existiert.
  - Korrekte `spawn_target`-Arrays setzen.
- [ ] Extraktion und Erstellung der `data/minecraft/worldgen/density_function/...` Dateien:
  - Definition aller referenzierten Density-Functions als eigenständige JSON-Dateien, um das Format 101 konform zu bedienen.
