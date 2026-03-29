# Airlock Monitor

Monitors airlock safety per airlock. Clone the template for each airlock in your base. Each instance monitors its own pressure sensors and doors, drives local displays, and reports alerts independently. BOD Core takes the worst-case across all airlocks.

## IC Housing Label

`BOD-LockMon` (all instances share this label)

## Hardware (per airlock)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-LockMon | -- | Runs a clone of this template |
| Gas Sensor(s) | *(your airlock name)* | -- | Input: airlock pressure |
| Door(s) | *(your airlock name)* | -- | Input: door open/closed state |
| LED Display (Small) | *(your airlock name)-Pr* | 0 (number) | Airlock pressure in kPa |
| LED Display (Small) | *(your airlock name)-Dr* | 0 (number) | Doors currently open |

Each airlock gets its own displays. Label them uniquely so they don't conflict.

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Airlock Pressure (kPa) | 20-120 | 5-20 / 120-150 | <5 / >150 |
| Doors Open | 0 | 1 | >1 |

### Alert Logic

- Low pressure suggests a leak or incomplete cycle
- High pressure suggests overpressure
- 1 open door is normal cycling (orange); 2+ open simultaneously could mean a breach (red)

## Setup

1. Clone `airlock-template.ic10` for each airlock
2. Edit the `EDIT` section at the top of each clone:
   - `SensorName`: Set to `HASH("YourAirlockName")` matching your gas sensor labels
   - `DoorName`: Set to `HASH("YourAirlockName")` matching your door labels (usually same name)
   - `DispPressure` / `DispDoor`: Set unique display labels per airlock
3. Label gas sensors and doors in each airlock with the matching name
4. Place LED displays near the airlock and label them to match
5. Label the IC Housing as `BOD-LockMon`
6. Wire to the data network

### Example: Main Airlock

```
define SensorName HASH("MainLock")
define DoorName HASH("MainLock")
define DispPressure HASH("Main-LkPr")
define DispDoor HASH("Main-Door")
```
Label gas sensors and doors in your main airlock as `MainLock`, displays as `Main-LkPr` and `Main-Door`.

### Example: Mining Airlock

```
define SensorName HASH("MineLock")
define DoorName HASH("MineLock")
define DispPressure HASH("Mine-LkPr")
define DispDoor HASH("Mine-Door")
```

### Example: Minimal (no local displays)

Skip placing displays if you only want alerting. The `sbn` writes harmlessly target non-existent devices. The alert still reports to BOD Core.

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-LockMon") Setting Maximum`
- Maximum aggregation means BOD Core sees the worst-case airlock
- Alert is the worst-case across pressure and door count within the airlock

## Fail-Silent Behavior

If no airlock monitors are deployed, `lbn` returns 0, which maps to alert level 0 (OK). No false alerts.
