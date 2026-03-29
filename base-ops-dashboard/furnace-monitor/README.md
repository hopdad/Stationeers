# Furnace Monitor

Monitors Arc Furnaces and Advanced Furnaces for dangerous temperature and pressure levels. Reads both furnace types and reports the worst-case values.

## IC Housing Label

`BOD-FurnMon`

## Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-FurnMon | -- | Runs this script |
| LED Display (Small) | BOD-Furn | 0 (number) | Max furnace temperature (K) |
| LED Display (Small) | BOD-FPrs | 0 (number) | Max furnace pressure (kPa) |

Furnaces (Arc + Advanced) are read by PrefabHash -- no labeling needed. The script takes the **maximum** temperature and pressure across both furnace types.

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Temperature (K) | <4000 | 4000-6000 | >6000 |
| Pressure (Pa) | <50000 | 50000-80000 | >80000 |

Note: Pressure is read in Pascals internally but displayed in kPa on BOD-FPrs.

## Setup

1. Build your Arc and/or Advanced Furnaces (no labeling needed)
2. Place LED displays near your smelting area
3. Label displays: `BOD-Furn` (Mode 0), `BOD-FPrs` (Mode 0)
4. Load `furnace-monitor.ic10` into an IC Housing labeled `BOD-FurnMon`
5. Wire everything to the same data network

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-FurnMon") Setting Average`
- Alert is the worst-case across temperature and pressure
