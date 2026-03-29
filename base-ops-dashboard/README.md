# Base Ops Dashboard (BOD)

A modular monitoring system for Stationeers. Each sub-module owns a subsystem -- reads its sensors, drives local displays with color-coded alerts, and reports status to a central aggregator that drives a master alert display.

## Architecture

```
atmo-monitor/ ────┐  (1..N zones)
power-monitor/ ───┤
solar-tracker/ ───┤
farm-monitor/ ────┼──> core/bod-core ──> Master Alert Display
airlock-monitor/ ─┤  (1..N airlocks)
furnace-monitor/ ─┤
storage-monitor/ ─┘  (1..N areas)
```

Each monitor writes an alert level to its IC Housing's `db.Setting`. BOD Core polls all monitors via `lbn` and rolls up the worst-case into a master alert (0=OK, 1=WARN, 2=CRIT).

## Modules

| Folder | IC Housing | Script | Purpose |
|--------|-----------|--------|---------|
| `core/` | BOD-Core | bod-core.ic10 | Aggregator: master alert + storage overview |
| `core/` | BOD-CycleMon | cycle-display.ic10 | Optional: rotating multi-reading display |
| `core/` | BOD-Alarm | alarm-plugin.ic10 | Optional: audible alarm on critical |
| `atmo-monitor/` | BOD-AtmoMon | atmo-zone-template.ic10 | Atmosphere: clone per zone, temp/press/O2/CO2/N2O |
| `power-monitor/` | BOD-PwrMon | power-monitor.ic10 | Power: battery, generation, solar angle |
| `solar-tracker/` | BOD-SolarMon | solar-tracker.ic10 | Solar panel tracking + efficiency |
| `farm-monitor/` | BOD-FarmMon | farm-monitor.ic10 | Farming: growth, occupancy, water pressure |
| `airlock-monitor/` | BOD-LockMon | airlock-template.ic10 | Airlocks: clone per airlock, pressure/doors |
| `furnace-monitor/` | BOD-FurnMon | furnace-monitor.ic10 | Furnaces: temp, pressure |
| `storage-monitor/` | BOD-StorMon | storage-monitor-template.ic10 | Storage: clone per area, fill % |

Each module folder has its own README with hardware BOM, thresholds, and setup instructions.

## Interface Contract

All monitors follow the same pattern:

1. **Read** sensors via `lbn` (named batch read) or `lb` (PrefabHash batch read)
2. **Drive** local displays with color-coded values (Green=OK, Orange=WARN, Red=CRIT)
3. **Write** alert level to IC Housing: `s db Setting <alert>`

Alert levels:
- `0` = OK (all green)
- `1` = WARN (at least one orange)
- `2` = CRIT (at least one red)

**Exception**: Storage monitors write fill percentage (0-100) instead of alert level. BOD Core evaluates storage thresholds itself.

BOD Core reads each monitor via `lbn`. Multi-instance modules (atmo, airlock, storage) use `Maximum` to surface the worst-case. Single-instance modules use `Average`.

## Color Codes

| Color | Value | Meaning |
|-------|-------|---------|
| Green | 2 | OK |
| Orange | 3 | Warning |
| Red | 4 | Critical |
| Blue | 6 | Night / suppressed |

Alert derivation trick: `sub vAlert vColor ColorGreen` maps Green->0, Orange->1, Red->2.

## Fail-Silent Behavior

Monitors that aren't deployed don't trigger false alerts:
- `lbn` returns 0 for missing IC Housings, which maps to alert level 0 (OK)
- Storage monitors use an explicit device count (`lbn ... On Sum`) -- BOD Core skips storage alert evaluation when count is zero

Deploy only the modules you need. The system works with any subset.

## Quick Start

1. Pick the modules you want to deploy
2. Read each module's README for its hardware BOM and setup steps
3. Label all devices with exact names from the BOM tables
4. Wire everything to a single data network with power
5. Load scripts into their IC Housings via a Computer
6. Deploy `core/bod-core.ic10` last -- it auto-discovers all active monitors

## Adding New Modules

Follow the pattern:

1. Create a new IC10 script that reads sensors, drives displays, and writes alert (0/1/2) to `s db Setting`
2. Label its IC Housing with a unique name (e.g., `BOD-NewMon`)
3. In `core/bod-core.ic10`, add one `lbn` line in `PollMonitors` and one `max` line in `EvalMaster`
4. Create a folder with the script and a README
5. BOD Core has 49 free lines for expansion

## Hardware Totals (full deployment)

- 8 IC Housings (minimum: 1 Core + modules you deploy)
- 3+ Gas Sensors (atmosphere, airlock, water)
- 1 Daylight Sensor
- 18 LED Displays
- 2 Diode Slides
- 1 Speaker (optional, alarm plugin)
