# BOD Core + Plugins

Central aggregator and optional plugins for the Base Ops Dashboard. This is your command center -- deploy these at your main base console.

## Scripts

| Script | IC Housing | Lines | Purpose |
|--------|-----------|-------|---------|
| `bod-core.ic10` | BOD-Core | 84 | Aggregator: polls all monitors, drives master alert |
| `cycle-display.ic10` | BOD-CycleMon | 58* | Optional: rotating multi-reading display |
| `alarm-plugin.ic10` | BOD-Alarm | 15 | Optional: audible alarm on critical alerts |
| `watchdog.ic10` | BOD-Watch | 119 | Optional: monitors BOD system health |

## BOD Core

### IC Housing Label

`BOD-Core`

### Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-Core | -- | Runs bod-core.ic10 |
| LED Display (Small) | BOD-Stor | 1 (percent) | Base-wide storage overview (avg fill) |
| LED Display (Small) | BOD-Alert | 0 (number) | Master alert level (0/1/2) |

### How It Works

BOD Core polls 7 subsystem monitors each tick via `lbn`:
- AtmoMon, LockMon, PwrMon, WasteMon (multi-instance, read via `Maximum` for worst-case)
- FarmMon, FurnMon, SolarMon, Watch (single-instance, read via `Average`)
- StorMon (multi-instance, fill percentage 0-100, evaluated against its own thresholds)

The master alert is the **worst-case** (maximum) across all monitors. Storage uses the **minimum** fill across all storage monitors for alert evaluation.

### Storage Thresholds (at Core level)

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Min Fill % | >25% | 10-25% | <10% |

### Fail-Silent Behavior

Monitors that aren't deployed don't trigger alerts. `lbn` returns 0 for missing devices, which maps to OK. Storage monitors use an explicit device count check (`lbn ... On Sum`) -- if no storage monitors exist, storage alerts are skipped.

### Adding New Monitors

1. Create your monitor script (see any existing module for the pattern)
2. Label its IC Housing `BOD-YourMon`
3. Add one `lbn` line in `PollMonitors` to read the new monitor's alert
4. Add one `max` line in `EvalMaster` to include it in the master alert
5. BOD Core has room for expansion (79/128 lines used)

---

## Cycling Display (Optional)

Consolidates multiple readings onto a single LED that rotates on a timer. Useful when wall space is limited.

### IC Housing Label

`BOD-CycleMon`

### Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-CycleMon | -- | Runs cycle-display.ic10 |
| LED Display | BOD-Cycle | (auto) | Rotating reading display |
| LED Display | BOD-Page | 0 (number) | Current page indicator |

### Configuration

Edit `cycle-display.ic10` before loading:

1. Uncomment the page blocks for readings you want in the rotation
2. Uncomment the matching `beq` jump table entry for each page
3. Set `CYCLE_COUNT` to the number of active pages
4. Set `CYCLE_TICKS` for rotation speed (default 10 = ~5 seconds per reading)

Ships with 5 active pages (Temp, Press, O2, Battery, Gen) and 13 more commented out. The display auto-switches Mode (number/percent/watts) per reading.

---

## Alarm Plugin (Optional)

Audible alarm that sounds when BOD Core's master alert hits critical.

### IC Housing Label

`BOD-Alarm`

### Hardware

| Device | Label | Purpose |
|--------|-------|---------|
| IC Housing | BOD-Alarm | Runs alarm-plugin.ic10 |
| Speaker | BOD-Siren | Audible alarm output |

### Configuration

Edit `alarm-plugin.ic10`:
- `ALARM_LEVEL`: `2` for critical-only (default), `1` for warn+crit
- `ALARM_DELAY`: Ticks before alarm triggers (default 5 = ~2.5 seconds)

The alarm uses a consecutive-tick debounce -- the alert must sustain for `ALARM_DELAY` ticks before the speaker activates. Auto-silences when the alert drops below threshold.

---

## Network Health Watchdog (Optional)

Monitors the BOD system itself. Counts active IC Housings per module label and alerts if any go offline (power loss, broken circuit, removed). Catches the one failure mode other monitors can't detect: a dead monitor can't report that it's dead.

### IC Housing Label

`BOD-Watch`

### Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-Watch | -- | Runs watchdog.ic10 |
| LED Display (Small) | BOD-Health | 0 (number) | Missing module count |

### Configuration

Edit `watchdog.ic10` -- set expected counts per module:

```
define ExpAtmo 1      # Number of atmo zone monitors
define ExpPwr 1       # Number of power grid monitors
define ExpSolar 1     # Solar tracker (usually 1)
define ExpFarm 1      # Farm monitors
define ExpLock 1      # Airlock monitors
define ExpFurn 1      # Furnace monitors
define ExpStor 0      # Storage monitors (0 = don't check)
define ExpWaste 0     # Waste monitors (0 = don't check)
```

Set to `0` for modules you don't deploy. The watchdog only checks modules with expected count > 0.

### Alert Levels

| BOD-Health Display | Color | Meaning |
|-------------------|-------|---------|
| 0 | Green | All modules present |
| 1 | Orange | 1 module offline |
| 2+ | Red | Multiple modules offline |

### Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-Watch") Setting Average`
- Feeds into the master alert like any other monitor
