# Farm Monitor

Monitors hydroponics farming operations: crop growth readiness, tray occupancy, and water supply pressure. Place displays near your farm for at-a-glance status.

## IC Housing Label

`BOD-FarmMon`

## Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-FarmMon | -- | Runs this script |
| Hydroponics Tray(s) | BOD-Farm | -- | Input: batch-read growth and occupancy |
| Gas Sensor | BOD-Water | -- | Input: water pipe pressure |
| LED Display (Small) | BOD-Grow | 1 (percent) | Average growth readiness |
| LED Display (Small) | BOD-Occ | 1 (percent) | Tray occupancy ratio |
| LED Display (Small) | BOD-H2O | 0 (number) | Water pressure in kPa |

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Tray Occupancy | >50% | 25-50% | <25% |
| Growth Readiness | >10% | 0-10% | 0% |
| Water Pressure (kPa) | >50 | 20-50 | <20 |

## Setup

1. Label all hydroponics trays `BOD-Farm`
2. Place a Gas Sensor on the water supply pipe and label it `BOD-Water`
3. Place LED displays near your farm area
4. Label displays: `BOD-Grow` (Mode 1), `BOD-Occ` (Mode 1), `BOD-H2O` (Mode 0)
5. Load `farm-monitor.ic10` into an IC Housing labeled `BOD-FarmMon`
6. Wire everything to the same data network

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-FarmMon") Setting Average`
- Alert is the worst-case across growth, occupancy, and water pressure
