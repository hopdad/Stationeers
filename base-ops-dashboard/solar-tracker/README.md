# Solar Tracker

Active solar panel tracking and efficiency monitoring. Reads a daylight sensor to determine sun position, adjusts all named solar panels to follow the sun, and reports generation efficiency to BOD Core.

## IC Housing Label

`BOD-SolarMon`

## Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-SolarMon | -- | Runs this script |
| Daylight Sensor | BOD-Sun | -- | Input: solar angle (shared with power monitor) |
| Solar Panel(s) | BOD-Panel | -- | Controlled: vertical angle set by tracker |
| LED Display (Small) | BOD-Trk | 0 (number) | Current tracking angle |
| LED Display (Small) | BOD-Eff | 1 (percent) | Solar efficiency ratio |

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Efficiency | >50% | 25-50% | <25% |
| Night | -- | -- | Blue (alerts suppressed) |

### Night Mode

When solar angle drops below 5 degrees:
- Panels **park at 0 degrees**
- Displays turn **blue**
- Efficiency alerts are **suppressed** (0% output at night is expected)

## Setup

1. Label all solar panels you want tracked as `BOD-Panel`
2. Ensure a Daylight Sensor labeled `BOD-Sun` exists on the network (can be shared with Power Monitor)
3. Place LED displays near your solar array or command center
4. Label displays: `BOD-Trk` (Mode 0) and `BOD-Eff` (Mode 1)
5. Load `solar-tracker.ic10` into an IC Housing labeled `BOD-SolarMon`
6. Wire everything to the same data network

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-SolarMon") Setting Average`
- At night, alert is always 0 (OK) regardless of efficiency
