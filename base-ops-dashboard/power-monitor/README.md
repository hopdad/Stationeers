# Power Monitor

Monitors battery charge and power generation. Includes solar angle tracking with night-mode suppression to avoid false alerts when batteries drain overnight.

## IC Housing Label

`BOD-PwrMon`

## Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-PwrMon | -- | Runs this script |
| LED Display (Medium) | BOD-Pwr | 1 (percent) | Battery charge ratio |
| LED Display (Small) | BOD-Gen | 2 (watts) | Total power generation |
| Daylight Sensor | BOD-Sun | -- | Input: solar angle |
| LED Display (Small) | BOD-Sun | 0 (number) | Solar angle display |
| Diode Slide | BOD-Bar-Pwr | -- | Battery charge bar graph |

Batteries (Small + Large) are read by PrefabHash -- no labeling needed. Solar panels and wind turbines are also read by PrefabHash for generation totals.

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Battery % | >50% | 20-50% | <20% |
| Solar Angle | >5 (day) | -- | <5 (night, blue) |

### Night Mode

When solar angle drops below 5 degrees:
- BOD-Sun and BOD-Gen displays turn **blue**
- Battery alert is **capped at orange** (drain at night is expected)
- Critical battery alert is suppressed to prevent false alarms

## Setup

1. Place a Daylight Sensor and label it `BOD-Sun`
2. Place LED displays and Diode Slide at your command center or power room
3. Label displays using exact names from the hardware table
4. Set display modes: BOD-Pwr to Mode 1, BOD-Gen to Mode 2, BOD-Sun to Mode 0
5. Load `power-monitor.ic10` into an IC Housing labeled `BOD-PwrMon`
6. Wire everything to the same data network

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-PwrMon") Setting Average`
- Night suppression caps the alert at 1 (WARN) even if battery is critically low
