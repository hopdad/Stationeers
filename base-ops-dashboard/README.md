# Base Ops Dashboard (BOD)

A micro-architecture monitoring system for Stationeers. Multiple IC10 scripts each own a subsystem and report status to a central aggregator that drives a master alert display.

## Architecture

```
Atmo Monitor ────┐
Power Monitor ───┤
StorMon #1..N ───┤
Farm Monitor ────┼──> BOD Core ──> Master Alert Display
Airlock Monitor ─┤
Furnace Monitor ─┘
```

Each monitor reads its sensors, drives its own displays with color-coded thresholds, and writes an alert level (0=OK, 1=WARN, 2=CRIT) to its IC Housing's `db.Setting`. BOD Core polls all monitors and rolls up the worst-case into a master alert.

## Scripts

| Script | IC Housing Label | Lines | Purpose |
|--------|-----------------|-------|---------|
| `atmo-monitor.ic10` | BOD-AtmoMon | 113 | Atmosphere: temp, pressure, O2, CO2, pollutants |
| `power-monitor.ic10` | BOD-PwrMon | 78 | Power: battery, generation, solar angle + night mode |
| `storage-monitor-template.ic10` | BOD-StorMon | 26 | Storage: clone per area, fill % |
| `farm-monitor.ic10` | BOD-FarmMon | 72 | Farming: growth, occupancy, water pressure |
| `airlock-monitor.ic10` | BOD-LockMon | 63 | Airlocks: pressure, door states |
| `furnace-monitor.ic10` | BOD-FurnMon | 55 | Furnaces: temp, pressure (Arc + Advanced) |
| `bod-core.ic10` | BOD-Core | 75 | Aggregator: master alert + storage overview |
| `cycle-display.ic10` | BOD-CycleMon | 58* | Configurable cycling display (*with 5 active pages) |

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
| LED Display (Small) | BOD-CO2 | 1 (percent) | Atmo Monitor |
| LED Display (Small) | BOD-Poll | 1 (percent) | Atmo Monitor |
| Diode Slide | BOD-Bar-O2 | — | Atmo Monitor |

### Power Monitor
| Device | Label | Display Mode | Driven By |
|--------|-------|-------------|-----------|
| IC Housing | BOD-PwrMon | — | power-monitor.ic10 |
| LED Display (Medium) | BOD-Pwr | 1 (percent) | Power Monitor |
| LED Display (Small) | BOD-Gen | 2 (watts) | Power Monitor |
| Daylight Sensor | BOD-Sun | — | Input (solar angle) |
| LED Display (Small) | BOD-Sun | 0 (number) | Power Monitor |
| Diode Slide | BOD-Bar-Pwr | — | Power Monitor |

### Farm Monitor
| Device | Label | Display Mode | Driven By |
|--------|-------|-------------|-----------|
| IC Housing | BOD-FarmMon | — | farm-monitor.ic10 |
| Hydroponics Tray(s) | BOD-Farm | — | Input (batch-read) |
| Gas Sensor | BOD-Water | — | Input (water pipe pressure) |
| LED Display (Small) | BOD-Grow | 1 (percent) | Farm Monitor |
| LED Display (Small) | BOD-Occ | 1 (percent) | Farm Monitor |
| LED Display (Small) | BOD-H2O | 0 (number) | Farm Monitor |

### Airlock Monitor
| Device | Label | Display Mode | Driven By |
|--------|-------|-------------|-----------|
| IC Housing | BOD-LockMon | — | airlock-monitor.ic10 |
| Gas Sensor(s) | BOD-Lock | — | Input (airlock atmosphere) |
| Door(s) | BOD-Door | — | Input (airlock doors) |
| LED Display (Small) | BOD-LkPr | 0 (number) | Airlock Monitor |
| LED Display (Small) | BOD-Door | 0 (number) | Airlock Monitor |

### Furnace Monitor
| Device | Label | Display Mode | Driven By |
|--------|-------|-------------|-----------|
| IC Housing | BOD-FurnMon | — | furnace-monitor.ic10 |
| LED Display (Small) | BOD-Furn | 0 (number) | Furnace Monitor |
| LED Display (Small) | BOD-FPrs | 0 (number) | Furnace Monitor |

### Per Storage Area
| Device | Label | Driven By |
|--------|-------|-----------|
| IC Housing | BOD-StorMon | storage-monitor-template.ic10 |

**Totals**: 7 IC Housings (min), 3+ Gas Sensors, 1 Daylight Sensor, 16 LED Displays, 2 Diode Slides

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
| CO2 Ratio | <0.02 | 0.02–0.05 | >0.05 |
| Pollutant (N2O) | <0.01 | 0.01–0.03 | >0.03 |

### Power (power-monitor.ic10)
| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Battery % | >50% | 20–50% | <20% |
| Solar Angle | >5° (day) | — | <5° (night, blue) |

Night mode: When solar angle < 5°, BOD-Sun and BOD-Gen displays turn blue. Battery alerts are capped at orange during night (drain is expected).

### Storage (bod-core.ic10)
| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Min Fill % | >25% | 10–25% | <10% |

### Farming (farm-monitor.ic10)
| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Tray Occupancy | >50% | 25–50% | <25% |
| Growth Readiness | >10% | 0–10% | 0% |
| Water Pressure (kPa) | >50 | 20–50 | <20 |

### Airlocks (airlock-monitor.ic10)
| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Airlock Pressure (kPa) | 20–120 | 5–20 / 120–150 | <5 / >150 |
| Doors Open | 0 | 1 | >1 |

### Furnaces (furnace-monitor.ic10)
| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Furnace Temp (K) | <4000 | 4000–6000 | >6000 |
| Furnace Pressure (Pa) | <50000 | 50000–80000 | >80000 |

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

## Cycling Display Setup

The cycling display consolidates multiple readings onto a single LED that rotates on a timer. Useful when wall space is limited.

**Hardware**: 1 IC Housing (`BOD-CycleMon`), 1 LED Display (`BOD-Cycle`), 1 LED Display (`BOD-Page` — page indicator)

**Configuration** (edit `cycle-display.ic10` before loading):
1. Uncomment the page blocks for readings you want in the rotation
2. Uncomment the matching `beq` jump table entry for each page
3. Set `CYCLE_COUNT` to the number of active pages
4. Set `CYCLE_TICKS` for speed (default 10 = ~5 seconds per reading)

Ships with 5 active pages (Temp, Press, O2, Battery, Gen) and 11 more commented out. All 16 BOD readings are available as page blocks. The display auto-switches Mode (number/percent/watts) per reading.

**Note**: The cycling display reads values back from existing LED displays via `lbn`. If reading `Color` back doesn't work in your game version, hardcode color per page block instead.

## Adding New Subsystem Monitors

1. Create a new IC10 script that reads sensors, drives displays, and writes alert (0/1/2) to `s db Setting`
2. Label its IC Housing `BOD-NewMon`
3. In `bod-core.ic10`, add one `lbn` line in `PollMonitors` and one `max` line in `EvalMaster`
4. BOD Core has 53 free lines for expansion

## Fail-Silent Behavior

Subsystem monitors that aren't deployed don't trigger false alerts. When `lbn` finds no matching IC Housing, the alert value defaults to 0 (OK). Storage monitors use an explicit device count check (`lbn ... On Sum`) to skip alert evaluation when no monitors exist.
