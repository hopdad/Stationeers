# Base Ops Dashboard (BOD)

A micro-architecture monitoring system for Stationeers. Multiple IC10 scripts each own a subsystem and report status to a central aggregator that drives a master alert display.

## Architecture

```
Atmo Monitor ──┐
Power Monitor ─┼──> BOD Core ──> Master Alert Display
StorMon #1..N ─┘
```

Each monitor reads its sensors, drives its own displays with color-coded thresholds, and writes an alert level (0=OK, 1=WARN, 2=CRIT) to its IC Housing's `db.Setting`. BOD Core polls all monitors and rolls up the worst-case into a master alert.

## Scripts

| Script | IC Housing Label | Lines | Purpose |
|--------|-----------------|-------|---------|
| `atmo-monitor.ic10` | BOD-AtmoMon | 83 | Atmosphere: temp, pressure, O2 |
| `power-monitor.ic10` | BOD-PwrMon | 56 | Power: battery charge, generation |
| `storage-monitor-template.ic10` | BOD-StorMon | 26 | Storage: clone per area, fill % |
| `bod-core.ic10` | BOD-Core | 61 | Aggregator: master alert + storage overview |

## Hardware Bill of Materials

### BOD Core
| Device | Label | Display Mode | Driven By |
|--------|-------|-------------|-----------|
| IC Housing | BOD-Core | — | bod-core.ic10 |
| LED Display (Small) | BOD-Stor | 1 (percent) | BOD Core |
| LED Display (Small) | BOD-Alert | 0 (number) | BOD Core |

### Atmosphere Monitor
| Device | Label | Display Mode | Driven By |
|--------|-------|-------------|-----------|
| IC Housing | BOD-AtmoMon | — | atmo-monitor.ic10 |
| Gas Sensor (1+) | BOD-Atmo | — | Input (batch-averaged) |
| LED Display (Small) | BOD-Temp | 0 (number) | Atmo Monitor |
| LED Display (Small) | BOD-Press | 0 (number) | Atmo Monitor |
| LED Display (Small) | BOD-O2 | 1 (percent) | Atmo Monitor |
| Diode Slide | BOD-Bar-O2 | — | Atmo Monitor |

### Power Monitor
| Device | Label | Display Mode | Driven By |
|--------|-------|-------------|-----------|
| IC Housing | BOD-PwrMon | — | power-monitor.ic10 |
| LED Display (Medium) | BOD-Pwr | 1 (percent) | Power Monitor |
| LED Display (Small) | BOD-Gen | 2 (watts) | Power Monitor |
| Diode Slide | BOD-Bar-Pwr | — | Power Monitor |

### Per Storage Area
| Device | Label | Driven By |
|--------|-------|-----------|
| IC Housing | BOD-StorMon | storage-monitor-template.ic10 |

**Totals**: 4 IC Housings (min), 1+ Gas Sensors, 6 LED Displays, 2 Diode Slides

## Wall Layout

```
BOD-Temp     BOD-Pwr        BOD-Alert
22°C         87%            0 (OK)

BOD-Press    BOD-Bar-Pwr    BOD-Stor
101 kPa      ████████░░     72%

BOD-O2 / BOD-Bar-O2         BOD-Gen
21% + bar                   1200W
```

## Threshold Reference

### Atmosphere (atmo-monitor.ic10)
| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Temp (°C) | 18–26 | 10–18 / 26–35 | <10 / >35 |
| Pressure (kPa) | 90–120 | 60–90 / 120–150 | <60 / >150 |
| O2 Ratio | 0.18–0.24 | 0.14–0.18 / 0.24–0.30 | <0.14 / >0.30 |

### Power (power-monitor.ic10)
| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Battery % | >50% | 20–50% | <20% |

### Storage (bod-core.ic10)
| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Min Fill % | >25% | 10–25% | <10% |

## Setup Checklist

1. Print hardware: 4 IC Housings, 1+ Gas Sensors, 6 LED Displays, 2 Diode Slides
2. Wire everything to a single data network with power
3. Label all devices with the Labeler using exact names from the BOM tables
4. Set LED display modes: BOD-Temp/Press to Mode 0, BOD-O2/Pwr/Stor to Mode 1, BOD-Gen to Mode 2, BOD-Alert to Mode 0
5. Load each IC10 script into its corresponding IC Housing via a Computer
6. Verify readings with a Tablet (Configuration cartridge)
7. Tune thresholds: edit `define` constants at the top of each script

## Storage Monitor Setup

1. Clone `storage-monitor-template.ic10` for each storage area
2. Edit the two `lbn` lines to match your storage device type and label name
3. Label the IC Housing as `BOD-StorMon` (all monitors share this name)
4. BOD Core automatically picks up new monitors via batch read

## Adding New Subsystem Monitors

1. Create a new IC10 script that reads sensors, drives displays, and writes alert (0/1/2) to `s db Setting`
2. Label its IC Housing `BOD-NewMon`
3. In `bod-core.ic10`, add one `lbn` line in `PollMonitors` and one `max` line in `EvalMaster`
4. BOD Core has 67 free lines for expansion
