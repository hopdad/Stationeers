# Atmosphere Monitor

Monitors base atmosphere conditions per zone. Clone the template for each area you want to monitor (base interior, greenhouse, mining outpost, etc.). Each zone drives its own local displays and reports alerts independently. BOD Core takes the worst-case across all zones.

## IC Housing Label

`BOD-AtmoMon` (all instances share this label)

## Hardware (per zone)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-AtmoMon | -- | Runs a clone of this template |
| Gas Sensor (1+) | *(your zone name)* | -- | Input: batch-averaged atmosphere readings |
| LED Display (Small) | *(your zone name)-Temp* | 0 (number) | Temperature in Celsius |
| LED Display (Small) | *(your zone name)-Press* | 0 (number) | Pressure in kPa |
| LED Display (Small) | *(your zone name)-O2* | 1 (percent) | Oxygen ratio |
| LED Display (Small) | *(your zone name)-CO2* | 1 (percent) | CO2 ratio |
| LED Display (Small) | *(your zone name)-Poll* | 1 (percent) | Pollutant (N2O) ratio |
| Diode Slide | *(your zone name)-Bar-O2* | -- | O2 bar graph |

Each zone gets its own set of displays. Label them uniquely per zone so they don't conflict.

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Temp (C) | 18-26 | 10-18 / 26-35 | <10 / >35 |
| Pressure (kPa) | 90-120 | 60-90 / 120-150 | <60 / >150 |
| O2 Ratio | 0.18-0.24 | 0.14-0.18 / 0.24-0.30 | <0.14 / >0.30 |
| CO2 Ratio | <0.02 | 0.02-0.05 | >0.05 |
| Pollutant (N2O) | <0.01 | 0.01-0.03 | >0.03 |

All thresholds are configurable via `define` constants in the script. You can tune thresholds per zone (e.g., a greenhouse might tolerate higher CO2).

## Setup

1. Clone `atmo-zone-template.ic10` for each zone
2. Edit the `EDIT` section at the top of each clone:
   - `SensorName`: Set to `HASH("YourZoneName")` matching your gas sensor labels
   - `DispTemp` through `DispBarO2`: Set unique display labels per zone
3. Label gas sensors in each zone with the matching name
4. Place LED displays near the zone and label them to match your defines
5. Label the IC Housing as `BOD-AtmoMon`
6. Wire to the data network

### Example: Base Interior

```
define SensorName HASH("BaseAtmo")
define DispTemp HASH("Base-Temp")
define DispPress HASH("Base-Press")
define DispO2 HASH("Base-O2")
define DispCO2 HASH("Base-CO2")
define DispPoll HASH("Base-Poll")
define DispBarO2 HASH("Base-Bar-O2")
```
Label gas sensors in your base as `BaseAtmo`, displays as `Base-Temp`, etc.

### Example: Greenhouse

```
define SensorName HASH("GHAtmo")
define DispTemp HASH("GH-Temp")
define DispPress HASH("GH-Press")
define DispO2 HASH("GH-O2")
define DispCO2 HASH("GH-CO2")
define DispPoll HASH("GH-Poll")
define DispBarO2 HASH("GH-Bar-O2")
```

### Example: Minimal (no local displays)

If you only want alerting without local displays, you can skip placing displays entirely. The `sbn` writes will harmlessly target non-existent devices. The alert still reports to BOD Core.

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-AtmoMon") Setting Maximum`
- Maximum aggregation means BOD Core sees the worst-case zone
- Alert is derived from the worst-case color across all 5 metrics within the zone

## Fail-Silent Behavior

If no atmosphere monitors are deployed, `lbn` returns 0, which maps to alert level 0 (OK). No false alerts.
