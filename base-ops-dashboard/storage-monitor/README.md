# Storage Monitor

Template script for monitoring storage areas. Clone this script for each storage area you want to track. Each instance drives a local fill-percentage display and reports to BOD Core for base-wide storage overview.

## IC Housing Label

`BOD-StorMon` (all instances share this label)

## Hardware (per storage area)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-StorMon | -- | Runs a clone of this template |
| LED Display (Small) | BOD-Fill | 1 (percent) | Local fill percentage display |

The storage devices themselves (Vending Machines, Fridges, Silos, Canisters, etc.) are read by name via `lbn` -- edit the two `lbn` lines in the script to match your device type and label.

## Thresholds (local display)

| Metric | Green | Orange | Red |
|--------|-------|--------|-----|
| Fill % | >25% | 10-25% | <10% |

BOD Core applies its own thresholds to the aggregated minimum fill across all storage monitors.

## Setup

1. Clone `storage-monitor-template.ic10` for each storage area
2. Edit the two `lbn` lines to match your storage:
   - **Slot-based** (Vending Machine, Fridge): use `HASH("StructureVendingMachine")` etc.
   - **Bulk** (Silo, Canister): use `HASH("StructureSilo")` etc.
   - Change `HASH("MyStorage")` to match your device's label name
3. Label the IC Housing as `BOD-StorMon`
4. Place an LED display labeled `BOD-Fill` near the storage area (Mode 1)
5. Wire to the data network

### Example: Food Vending Machine

```
lbn vCount HASH("StructureVendingMachine") HASH("FoodStorage") Quantity Sum
lbn vMax HASH("StructureVendingMachine") HASH("FoodStorage") Maximum Sum
```

### Example: Ore Silo

```
lbn vCount HASH("StructureSilo") HASH("OreSilo") Quantity Sum
lbn vMax HASH("StructureSilo") HASH("OreSilo") Maximum Sum
```

## Interface Contract

- **Writes to `db.Setting`**: Fill percentage (0-100)
- **BOD Core reads**: Average for overview display, Minimum for alert evaluation
- Multiple storage monitors all write to their own IC Housing's `db.Setting`
- BOD Core uses `lbn ... HASH("BOD-StorMon") Setting Average` and `... Minimum` to aggregate

## Fail-Silent Behavior

If no storage monitors are deployed, BOD Core detects zero devices via `lbn ... On Sum` and skips storage alert evaluation entirely. No false alerts.
