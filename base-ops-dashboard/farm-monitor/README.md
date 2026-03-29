# Farm Monitor

Monitors hydroponics farming per zone: crop growth, tray occupancy, and water supply pressure. Clone the template for each farm area (food farm, herb farm, separate greenhouses). Place displays near each farm for at-a-glance status.

## IC Housing Label

`BOD-FarmMon` (all instances share this label)

## Hardware (per zone)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-FarmMon | -- | Runs a clone of this template |
| Hydroponics Tray(s) | *(your zone name)* | -- | Input: growth, occupancy |
| Gas Sensor | *(your zone name)* | -- | Input: water pipe pressure |
| LED Display (Small) | *(per zone)* | 1 (percent) | Average growth readiness |
| LED Display (Small) | *(per zone)* | 1 (percent) | Tray occupancy ratio |
| LED Display (Small) | *(per zone)* | 0 (number) | Water pressure in kPa |

Label trays and the water pipe gas sensor with the same name per zone.

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Tray Occupancy | >50% | 25-50% | <25% |
| Growth Readiness | >10% | 0-10% | 0% |
| Water Pressure (kPa) | >50 | 20-50 | <20 |

## Setup

1. Clone `farm-zone-template.ic10` per farm zone
2. Edit the `EDIT` section:
   - `TrayName`: `HASH("YourFarmName")` matching your tray labels
   - `WaterName`: `HASH("YourFarmName")` matching your water sensor label (usually same)
   - `DispGrow/DispOcc/DispH2O`: Unique display labels per zone
3. Label trays and water sensor in each zone
4. Place and label displays
5. Label IC Housing as `BOD-FarmMon`

### Example: Food Farm

```
define TrayName HASH("FoodFarm")
define WaterName HASH("FoodFarm")
define DispGrow HASH("Food-Grow")
define DispOcc HASH("Food-Occ")
define DispH2O HASH("Food-H2O")
```

### Example: Herb Garden

```
define TrayName HASH("HerbFarm")
define WaterName HASH("HerbFarm")
define DispGrow HASH("Herb-Grow")
define DispOcc HASH("Herb-Occ")
define DispH2O HASH("Herb-H2O")
```

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-FarmMon") Setting Maximum`
- Maximum aggregation surfaces the worst-case farm zone

## Fail-Silent Behavior

If no farm monitors are deployed, `lbn` returns 0 (OK). No false alerts.
