# Power Monitor

Monitors battery charge and power generation per grid. Clone the template for each independent power grid. Includes an optional circuit breaker for automatic load shedding when batteries run low.

## Scripts

| Script | IC Housing | Lines | Purpose |
|--------|-----------|-------|---------|
| `power-grid-template.ic10` | BOD-PwrMon | 86 | Battery + generation monitor per grid |
| `circuit-breaker.ic10` | BOD-Breaker | 87 | Optional: automatic load shedding |

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
