# Paper-Kompatibilität

## Stand: Minecraft 26.1.2 / Paper 1.21.x

## Bekanntes Problem: Chunk-Loading über Y=319

Paper hat in Version 1.21.x bekannte Probleme mit dem Laden von Chunks
oberhalb der Vanilla-Grenze von Y=319:

- **Symptom**: Chunks über Y=319 werden nicht geladen / bleiben unsichtbar
- **Ursache**: Hardcodierte Limits in Paper's Netcode / Chunk-Serialisierung
- **Status**: Muss mit 26.1.2 getestet werden – neuere Paper-Versionen haben Fixes

### Workaround (falls Problem auftritt)

1. Testen ob Fabric/Vanilla-Server das Problem hat (Baseline)
2. Paper GitHub Issues prüfen: https://github.com/PaperMC/Paper/issues
3. Als temporären Workaround: `logical_height` auf `320` belassen,
   physische `height` auf 2064 – dann können nur Datapack-Mechaniken
   die volle Höhe nutzen, Spieler aber nicht normal bauen

## Empfohlene Paper-Konfiguration

Folgende Werte in `config/paper-world-defaults.yml` auf dem Server anpassen
(diese Datei gehört NICHT in dieses Repo):

```yaml
chunks:
  # Reduzieren wegen größerer Chunks
  max-auto-save-chunks-per-tick: 6
  
mobs:
  # Spawning-Limits anpassen da mehr vertikaler Raum
  monster-spawn-max-light-level: 7

entity:
  # Mehr Subchunks = mehr Entity-Tracking-Overhead
  tracking-range-y: false  # oder explizit setzen falls verfügbar
```

In `server.properties`:
```properties
# Render Distance reduzieren (Chunks sind ~5x größer)
view-distance=8
simulation-distance=6
```

## Getestete Versionen

| Minecraft | Paper Build | Ergebnis | Tester | Datum |
|---|---|---|---|---|
| 26.1.2 | – | 🔲 Noch nicht getestet | – | – |

## Paper Issues (relevant)

- Chunk height limit: https://github.com/PaperMC/Paper/issues (suche: "height limit")
- World height datapack: https://github.com/PaperMC/Paper/issues (suche: "dimension type")
