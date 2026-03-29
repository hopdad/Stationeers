# Waste Processing Monitor

Monitors filtration units and waste gas line pressure. Detects offline filters and pipe backups before they cause atmosphere contamination. Clone per waste processing area.

## IC Housing Label

`BOD-WasteMon` (all instances share this label)

## Hardware (per waste area)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-WasteMon | -- | Runs a clone of this template |
| Gas Sensor | *(your area name)* | -- | Input: waste pipe pressure |
| LED Display (Small) | *(per area)* | 0 (number) | Active filtration unit count |
| LED Display (Small) | *(per area)* | 0 (number) | Waste line pressure (kPa) |

Filtration units (basic + advanced) are read by PrefabHash or by name if labeled. No labeling required for a single waste area -- the script counts all filtration units on the network. For multiple areas, label your filters with the area name.

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Missing Filters | 0 | 1 | >1 |
| Waste Pressure (kPa) | <500 | 500-800 | >800 |

### Alert Logic

- **Filter count**: Compares active `On` count against `ExpFilters` define. Missing units indicate power loss or breakdown.
- **Waste pressure**: High pressure on the waste pipe means filters can't keep up, risking backup into the base atmosphere.

## Configuration

Edit `waste-monitor-template.ic10`:

```
define FilterName HASH("MyWaste")    # Label on filtration units (for multi-area)
define SensorName HASH("MyWaste")    # Label on waste pipe gas sensor
define DispFilt HASH("BOD-Filt")     # Filter count display
define DispWPrs HASH("BOD-WPrs")     # Waste pressure display
define ExpFilters 2                  # How many filters you expect running
define PressWarnHi 500               # Pressure warn threshold (kPa)
define PressCritHi 800               # Pressure crit threshold (kPa)
```

## Setup

1. Clone `waste-monitor-template.ic10` per waste processing area
2. Edit the `EDIT` section with your area's sensor/display labels
3. Set `ExpFilters` to match the number of filtration units in the area
4. Place a gas sensor on the waste pipe and label it to match `SensorName`
5. (Optional) Label filtration units to match `FilterName` for multi-area setups
6. Place LED displays near the waste processing equipment
7. Label IC Housing as `BOD-WasteMon`

### Example: Main Base Waste Processing

```
define FilterName HASH("BaseWaste")
define SensorName HASH("BaseWaste")
define DispFilt HASH("Base-Filt")
define DispWPrs HASH("Base-WPrs")
define ExpFilters 3
```

### Example: Mining Outpost (single filter)

```
define FilterName HASH("MineWaste")
define SensorName HASH("MineWaste")
define DispFilt HASH("Mine-Filt")
define DispWPrs HASH("Mine-WPrs")
define ExpFilters 1
```

## Relationship to Atmosphere Monitor

The atmosphere monitor tracks the **symptoms** of waste gas problems (rising CO2, N2O). The waste monitor tracks the **equipment** doing the cleanup. Together they give you both early warning (equipment failure) and confirmation (atmosphere impact).

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-WasteMon") Setting Maximum`
- Maximum aggregation surfaces the worst-case waste processing area

## Fail-Silent Behavior

If no waste monitors are deployed, `lbn` returns 0 (OK). No false alerts.
