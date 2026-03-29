# Furnace Monitor

Monitors Arc and Advanced Furnace temperature and pressure per smelting area. Clone the template if you have furnaces in different locations. Place displays near each smelting station.

## IC Housing Label

`BOD-FurnMon` (all instances share this label)

## Hardware (per area)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-FurnMon | -- | Runs a clone of this template |
| Arc / Advanced Furnace(s) | *(your area name)* | -- | Input: temperature, pressure |
| LED Display (Small) | *(per area)* | 0 (number) | Max furnace temperature (K) |
| LED Display (Small) | *(per area)* | 0 (number) | Max furnace pressure (kPa) |

Label all furnaces in an area with the same name. Supports both `StructureArcFurnace` and `StructureAdvancedFurnace` — takes the worst-case across both types.

## Thresholds

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Temperature (K) | <4000 | 4000-6000 | >6000 |
| Pressure (Pa) | <50000 | 50000-80000 | >80000 |

## Setup

1. Clone `furnace-area-template.ic10` per smelting area
2. Edit the `EDIT` section:
   - `FurnName`: `HASH("YourAreaName")` matching your furnace labels
   - `DispTemp/DispPrs`: Unique display labels per area
3. Label furnaces and place displays
4. Label IC Housing as `BOD-FurnMon`

### Example: Main Smelter

```
define FurnName HASH("MainSmelt")
define DispTemp HASH("Main-Furn")
define DispPrs HASH("Main-FPrs")
```

### Example: Alloy Room

```
define FurnName HASH("AlloySmelt")
define DispTemp HASH("Alloy-Furn")
define DispPrs HASH("Alloy-FPrs")
```

## Interface Contract

- **Writes to `db.Setting`**: Alert level (0=OK, 1=WARN, 2=CRIT)
- **BOD Core reads**: `lbn ... HASH("BOD-FurnMon") Setting Maximum`
- Maximum aggregation surfaces the worst-case smelting area

## Fail-Silent Behavior

If no furnace monitors are deployed, `lbn` returns 0 (OK). No false alerts.
