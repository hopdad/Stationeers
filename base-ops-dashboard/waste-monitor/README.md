# Waste Processing Monitor

Monitors filtration units directly — no separate gas sensor needed. Tracks operational status, input pressure (gas backlog), and waste output pressure (tank filling up). Clone per waste processing area.

## IC Housing Label

`BOD-WasteMon` (all instances share this label)

## Hardware (per waste area)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-WasteMon | -- | Runs a clone of this template |
| Filtration Unit(s) | *(your area name)* | -- | Input: read PressureInput/PressureOutput2/On |
| LED Display (Small) | *(per area)* | 0 (number) | Active filtration unit count |
| LED Display (Small) | *(per area)* | 0 (number) | Input pressure (kPa) |
| LED Display (Small) | *(per area)* | 0 (number) | Waste output pressure (kPa) |

No gas sensor required — the script reads `PressureInput` and `PressureOutput2` directly from the filtration units via `lbn`. Supports both `StructureFiltration` and `StructureAdvancedFiltration`.

## What It Monitors

| Metric | What It Means | Why It Matters |
|--------|--------------|----------------|
| Filter Count | Active units vs expected | Offline filter = gas not being processed |
| Input Pressure | Pressure on the intake side | High = gas backing up, filters overwhelmed |
| Output Pressure | Pressure on waste output side | High = waste tank filling up, nowhere to dump |

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Missing Filters | 0 | 1 | >1 |
| Input Pressure (kPa) | <500 | 500-800 | >800 |
| Output Pressure (kPa) | <8000 | 8000-9500 | >9500 |

## Configuration

Edit `waste-monitor-template.ic10`:

```
# --- EDIT THESE PER AREA ---
define FilterName HASH("MyWaste")     # Label on your filtration units
define DispFilt HASH("BOD-Filt")      # Filter count display
define DispIn HASH("BOD-WIn")         # Input pressure display
define DispOut HASH("BOD-WOut")       # Waste output pressure display
# --- END EDIT ---
define ExpFilters 2                   # Expected active filter count
```

## Setup

1. Label your filtration units with a consistent name (e.g., `BaseWaste`)
2. Clone `waste-monitor-template.ic10` for each waste processing area
3. Edit the `EDIT` section with your area's filter label and display names
4. Set `ExpFilters` to match your unit count
5. Place LED displays near the filtration equipment
6. Label IC Housing as `BOD-WasteMon`

### Example: Main Base CO2 Processing

```
define FilterName HASH("BaseCO2")
define DispFilt HASH("Base-Filt")
define DispIn HASH("Base-WIn")
define DispOut HASH("Base-WOut")
define ExpFilters 3
```

### Example: Mining Outpost

```
define FilterName HASH("MineWaste")
define DispFilt HASH("Mine-Filt")
define DispIn HASH("Mine-WIn")
define DispOut HASH("Mine-WOut")
define ExpFilters 1
```

## Filtration Properties Used

| Property | Source | Batch Mode | Purpose |
|----------|--------|-----------|---------|
| `On` | Filtration units | Sum | Count active units |
| `PressureInput` | Filtration units | Maximum | Worst-case intake pressure |
| `PressureOutput2` | Filtration units | Maximum | Worst-case waste output |

Note: `PressureOutput2` is the **waste/secondary** output. `PressureOutput` is the filtered/primary output.

## Relationship to Atmosphere Monitor

| Monitor | Watches | Alert Timing |
|---------|---------|-------------|
| **Waste Monitor** | Filtration equipment health | **Early warning** — catches equipment failure before gas builds up |
| **Atmo Monitor** | Room atmosphere composition | **Late warning** — catches the symptom after gas has already risen |

Deploy both for defense in depth.

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-WasteMon") Setting Maximum`
- Maximum aggregation surfaces the worst-case waste processing area

## Fail-Silent Behavior

If no waste monitors are deployed, `lbn` returns 0 (OK). No false alerts.
