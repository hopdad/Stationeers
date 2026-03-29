# Power Monitor

Monitors battery charge and power generation per grid. Clone the template for each independent power grid. Includes optional companion scripts for load shedding, trend analysis, and auto-restart.

## Scripts

| Script | IC Housing | Lines | Purpose |
|--------|-----------|-------|---------|
| `power-grid-template.ic10` | BOD-PwrMon | 86 | Battery + generation monitor per grid |
| `circuit-breaker.ic10` | BOD-Breaker | 87 | Optional: automatic load shedding |
| `battery-trend.ic10` | BOD-Trend | 68 | Optional: charge rate trend display |
| `breaker-restart.ic10` | BOD-Restart | 82 | Optional: staged device restart after breaker |

---

## Power Grid Template

### IC Housing Label

`BOD-PwrMon` (all instances share this label)

### Hardware (per grid)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-PwrMon | -- | Runs a clone of this template |
| Battery (Small/Large) | *(your grid name)* | -- | Input: charge + capacity |
| Solar Panel(s) | *(your grid name)* | -- | Input: current generation |
| Wind Turbine(s) | *(your grid name)* | -- | Input: current generation |
| Daylight Sensor | BOD-Sun | -- | Input: solar angle (shared across grids) |
| LED Display (Small) | *(per grid)* | 1 (percent) | Battery charge ratio |
| LED Display (Small) | *(per grid)* | 2 (watts) | Total power generation |
| LED Display (Small) | *(per grid)* | 0 (number) | Solar angle |
| Diode Slide | *(per grid)* | -- | Battery charge bar graph |

Label **all** batteries, solar panels, and turbines on a grid with the same name. `lbn` reads by PrefabHash + Name, so one name works across device types.

### Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Battery % | >50% | 20-50% | <20% |

### Night Mode

When solar angle < 5 degrees:
- Solar and generation displays turn **blue**
- Battery alert **capped at orange** (drain at night is expected)

### Setup

1. Clone `power-grid-template.ic10` for each power grid
2. Edit the `EDIT` section:
   - `GridName`: Set to `HASH("YourGridName")` -- label all batteries + generators with this
   - `SunName`: Daylight sensor label (default `BOD-Sun`, shared across grids)
   - `DispPwr/DispGen/DispSun/DispBarPwr`: Unique display labels per grid
3. Label all batteries, solar panels, and turbines on the grid with your grid name
4. Place displays and label them to match
5. Label IC Housing as `BOD-PwrMon`

### Example: Main Base Grid

```
define GridName HASH("MainGrid")
define SunName HASH("BOD-Sun")
define DispPwr HASH("Main-Pwr")
define DispGen HASH("Main-Gen")
define DispSun HASH("BOD-Sun")
define DispBarPwr HASH("Main-Bar-Pwr")
```
Label all main base batteries and generators as `MainGrid`.

### Example: Outpost Grid

```
define GridName HASH("Outpost")
define SunName HASH("BOD-Sun")
define DispPwr HASH("Out-Pwr")
define DispGen HASH("Out-Gen")
define DispSun HASH("Out-Sun")
define DispBarPwr HASH("Out-Bar-Pwr")
```

### Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-PwrMon") Setting Maximum`
- Maximum aggregation surfaces the worst-case grid

---

## Circuit Breaker (Optional)

Automatic load shedding when battery charge drops too low. Shuts down non-critical systems in priority order and restores them when charge recovers.

### IC Housing Label

`BOD-Breaker`

### Concept

Route power to different base sections through control devices (Transformers, APCs, etc.). Label them by priority tier:

```
Critical (life support, airlocks) ─── No label, always powered
Tier 2 (farms, furnaces) ──────────── "Shed-T2" transformers
Tier 3 (lights, displays) ─────────── "Shed-T3" transformers
```

When battery charge drops:
1. **Below 40%**: Tier 3 sheds (luxury systems off)
2. **Below 20%**: Tier 2 sheds (important systems off)
3. Critical systems stay powered throughout

Recovery uses **hysteresis** to prevent oscillation:
- Tier 3 restores at **55%** (not 40%)
- Tier 2 restores at **35%** (not 20%)

### Hardware

| Device | Label | Purpose |
|--------|-------|---------|
| IC Housing | BOD-Breaker | Runs circuit-breaker.ic10 |
| Transformer(s) | Shed-T3 | Controls power to tier 3 (luxury) |
| Transformer(s) | Shed-T2 | Controls power to tier 2 (important) |
| LED Display (Small) | BOD-Shed | Tiers shed count (0/1/2) |

### Configuration

Edit `circuit-breaker.ic10`:
- `GridName`: Match the power-grid-template for this grid
- `CtrlType`: PrefabHash of your control device (default: `StructureTransformerSmall`)
- `Tier3Name` / `Tier2Name`: Labels on your control devices
- Shed/restore thresholds: Adjust the 4 threshold defines

### Thresholds

| Event | Default | Configurable |
|-------|---------|-------------|
| Shed Tier 3 | Battery < 40% | `Tier3Shed` |
| Restore Tier 3 | Battery > 55% | `Tier3Restore` |
| Shed Tier 2 | Battery < 20% | `Tier2Shed` |
| Restore Tier 2 | Battery > 35% | `Tier2Restore` |

### Display

BOD-Shed shows how many tiers are currently shed:
- **0** (green): All systems running
- **1** (orange): Tier 3 shed (luxury off)
- **2** (red): Tier 2 + 3 shed (only critical systems running)

### Recommended Power Topology

```
Batteries ──┬── [Transformer "Shed-T3"] ── Lights, Displays, ...
             ├── [Transformer "Shed-T2"] ── Farms, Furnaces, ...
             └── (direct) ── Life Support, Airlocks, ...
```

The circuit breaker controls `On` on the labeled transformers. When a transformer is off, it stops passing power to everything downstream.

### Interface Contract

- **Writes to `db.Setting`**: Shed count (0/1/2)
- Not read by BOD Core by default (power alert already covers low-battery state)
- Add a `lbn` line in BOD Core if you want breaker status on the master display

---

## Battery Trend (Optional)

Tracks charge rate over time using an exponential moving average. Shows whether batteries are charging or draining, and how fast. Gives you early warning before the circuit breaker activates.

### IC Housing Label

`BOD-Trend`

### Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-Trend | -- | Runs battery-trend.ic10 |
| LED Display (Small) | *(per grid)* | 0 (number) | Charge rate (+ charging, - draining) |

### Configuration

```
define GridName HASH("MyGrid")        # Match power-grid-template
define DispTrend HASH("BOD-Trend")    # Display label
define DrainWarn -0.001               # Moderate drain rate
define DrainCrit -0.005               # Heavy drain rate
define SmoothAlpha 0.3                # EMA smoothing (0.2=gentle, 0.5=responsive)
```

### Display

- **Green**: Charging or stable
- **Orange**: Moderate drain (losing ~0.1%/tick)
- **Red**: Heavy drain (losing ~0.5%/tick)

The smoothed delta filters out per-tick noise so you see the actual trend, not momentary fluctuations.

---

## Breaker Auto-Restart (Optional)

Companion to the circuit breaker. Some devices (furnaces, hydro trays, filtration) don't resume cleanly when power is cut and restored. This script detects when the breaker restores a tier and re-enables device groups after a configurable delay to prevent inrush power spikes.

### IC Housing Label

`BOD-Restart`

### Hardware

| Device | Label | Purpose |
|--------|-------|---------|
| IC Housing | BOD-Restart | Runs breaker-restart.ic10 |

No displays — this is purely an actuator.

### Configuration

```
define CtrlType HASH("StructureTransformerSmall")  # Match circuit breaker
define Tier3Name HASH("Shed-T3")                   # Match circuit breaker
define Tier2Name HASH("Shed-T2")                   # Match circuit breaker
define Dev3Type HASH("StructureConsoleLED5")        # Devices to restart in tier 3
define Dev3Name HASH("Restart-T3")                  # Label on tier 3 restart devices
define Dev2Type HASH("StructureHydroponics")        # Devices to restart in tier 2
define Dev2Name HASH("Restart-T2")                  # Label on tier 2 restart devices
define RestartDelay 5                               # Ticks to wait before restarting
```

### How It Works

1. Watches the breaker's control devices for a rising edge (tier goes from off to on)
2. Waits `RestartDelay` ticks after restore (prevents inrush spike)
3. Writes `On 1` to the labeled restart device group
4. Returns to idle until the next breaker event

### Device Labeling

Label devices that need manual restart with the restart group name:
- Tier 3 devices needing restart → label as `Restart-T3`
- Tier 2 devices needing restart → label as `Restart-T2`

Devices that auto-resume after power returns don't need the restart label.
