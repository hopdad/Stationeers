# Atmosphere Monitor

Monitors base atmosphere conditions near gas sensors. Drives local displays with color-coded alerts and reports worst-case alert level to BOD Core.

## IC Housing Label

`BOD-AtmoMon`

## Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-AtmoMon | -- | Runs this script |
| Gas Sensor (1+) | BOD-Atmo | -- | Input: batch-averaged atmosphere readings |
| LED Display (Small) | BOD-Temp | 0 (number) | Temperature in Celsius |
| LED Display (Small) | BOD-Press | 0 (number) | Pressure in kPa |
| LED Display (Small) | BOD-O2 | 1 (percent) | Oxygen ratio |
| LED Display (Small) | BOD-CO2 | 1 (percent) | CO2 ratio |
| LED Display (Small) | BOD-Poll | 1 (percent) | Pollutant (N2O) ratio |
| Diode Slide | BOD-Bar-O2 | -- | O2 bar graph |

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Temp (C) | 18-26 | 10-18 / 26-35 | <10 / >35 |
| Pressure (kPa) | 90-120 | 60-90 / 120-150 | <60 / >150 |
| O2 Ratio | 0.18-0.24 | 0.14-0.18 / 0.24-0.30 | <0.14 / >0.30 |
| CO2 Ratio | <0.02 | 0.02-0.05 | >0.05 |
| Pollutant (N2O) | <0.01 | 0.01-0.03 | >0.03 |

All thresholds are configurable via `define` constants at the top of the script.

## Setup

1. Place gas sensors in the area you want to monitor
2. Label all gas sensors `BOD-Atmo` with the Labeler
3. Place LED displays and Diode Slide near the sensors
4. Label displays using exact names from the hardware table above
5. Set LED display modes: BOD-Temp and BOD-Press to Mode 0, BOD-O2/CO2/Poll to Mode 1
6. Load `atmo-monitor.ic10` into an IC Housing labeled `BOD-AtmoMon`
7. Wire everything to the same data network

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-AtmoMon") Setting Average`
- Alert level is derived from the worst-case color across all 5 metrics
