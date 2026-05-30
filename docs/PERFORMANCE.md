# Performance-Guide

## Warum hohe Welten teuer sind

Ein Minecraft-Chunk (16×16 Blöcke) wird vertikal in **Subchunks** à 16 Blöcken aufgeteilt.
Mit `height=2064` hat jeder Chunk **129 Subchunks** statt normalerweise 24.

Das bedeutet:
- ~5,4× mehr RAM pro Chunk
- ~5,4× mehr Disk-Schreiboperationen beim Speichern
- ~5,4× mehr Lighting-Berechnungen
- Signifikant höhere GC-Last auf der JVM

## Empfohlene JVM-Flags (Paper)

```bash
# Für Server mit 16 GB RAM, Paper 26.1.2
java -Xms8G -Xmx8G \
  -XX:+UseG1GC \
  -XX:+ParallelRefProcEnabled \
  -XX:MaxGCPauseMillis=200 \
  -XX:+UnlockExperimentalVMOptions \
  -XX:+DisableExplicitGC \
  -XX:+AlwaysPreTouch \
  -XX:G1NewSizePercent=30 \
  -XX:G1MaxNewSizePercent=40 \
  -XX:G1HeapRegionSize=8M \
  -XX:G1ReservePercent=20 \
  -XX:G1HeapWastePercent=5 \
  -XX:G1MixedGCCountTarget=4 \
  -XX:InitiatingHeapOccupancyPercent=15 \
  -XX:G1MixedGCLiveThresholdPercent=90 \
  -XX:G1RSetUpdatingPauseTimePercent=5 \
  -XX:SurvivorRatio=32 \
  -XX:+PerfDisableSharedMem \
  -XX:MaxTenuringThreshold=1 \
  -jar paper.jar --nogui
```

## Empfohlene server.properties

```properties
view-distance=8
simulation-distance=6
```

## Empfohlene paper-world-defaults.yml Anpassungen

```yaml
chunks:
  max-auto-save-chunks-per-tick: 6
  prevent-moving-into-unloaded-chunks: true

entities:
  spawning:
    monster-spawn-max-light-level: 7
```

## RAM-Schätzung

| Spieler | View Distance | Geschätzter RAM (Chunks im Speicher) |
|---|---|---|
| 1–5 | 8 | ~4–6 GB |
| 5–20 | 8 | ~8–12 GB |
| 20–50 | 6 | ~16–24 GB |

> Diese Werte sind Schätzungen. Reales Profiling auf dem Zielsystem empfohlen.

## Monitoring

- Paper Spark Plugin: `https://spark.lucko.me/` – für JVM/TPS Profiling
- `/paper heap` Command – Heap-Nutzung direkt in Paper
- Ziel: TPS ≥ 19.5 unter Last, MSPT < 45ms
