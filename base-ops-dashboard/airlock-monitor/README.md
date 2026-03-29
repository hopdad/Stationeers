# Airlock Monitor

Monitors airlock safety: pressure levels and door states. Detects leaks, overpressure, and potential breaches when multiple doors are open simultaneously.

## IC Housing Label

`BOD-LockMon`

## Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-LockMon | -- | Runs this script |
| Gas Sensor(s) | BOD-Lock | -- | Input: airlock atmosphere pressure |
| Door(s) | BOD-Door | -- | Input: door open/closed state |
| LED Display (Small) | BOD-LkPr | 0 (number) | Airlock pressure in kPa |
| LED Display (Small) | BOD-Door | 0 (number) | Number of doors currently open |

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Airlock Pressure (kPa) | 20-120 | 5-20 / 120-150 | <5 / >150 |
| Doors Open | 0 | 1 | >1 |

### Alert Logic

- Pressure uses dual thresholds: low pressure suggests a leak, high pressure suggests overpressure
- Door count: 1 open door is normal cycling (orange), 2+ open doors could mean a breach (red)

## Setup

1. Place gas sensors inside airlock chambers and label them `BOD-Lock`
2. Label airlock doors `BOD-Door`
3. Place LED displays near your airlock area or command center
4. Label displays: `BOD-LkPr` (Mode 0), `BOD-Door` (Mode 0)
5. Load `airlock-monitor.ic10` into an IC Housing labeled `BOD-LockMon`
6. Wire everything to the same data network

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-LockMon") Setting Average`
- Alert is the worst-case across pressure and door count
