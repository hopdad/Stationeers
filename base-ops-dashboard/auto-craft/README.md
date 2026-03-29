# Auto-Craft System

Automated production management for Stationeers crafting machines. Monitors item stock via `ExportCount`, triggers crafting when supplies run low. Independent micro-architecture with its own aggregator — optionally feeds into the BOD alert network.

## Architecture

```
auto-craft-template ×N ──> craft-core ──> Craft Status Displays
  (each manages one              ↓
   machine + recipe)      (optional) BOD network via db.Setting
```

## Scripts

| Script | IC Housing | Lines | Purpose |
|--------|-----------|-------|---------|
| `auto-craft-template.ic10` | BOD-Craft | 80 | Clone per machine/recipe — production controller |
| `craft-core.ic10` | BOD-CraftCore | 74 | Aggregator: polls all crafters, master status |

---

## Auto-Craft Template

One IC per machine/recipe combo. Watches the machine's `ExportCount`, triggers crafting when below target.

### IC Housing Label

`BOD-Craft` (all instances share this label)

### Hardware (per machine)

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-Craft | -- | Runs a clone of this template |
| Crafting Machine | *(your machine name)* | -- | Controlled: recipe + activate |
| LED Display (Small) | *(per machine)* | 0 (number) | Current stock count |

### Configuration

```
define MachineType HASH("StructureAutolathe")   # Machine PrefabHash
define MachineName HASH("MyCrafter")            # Label on the machine
define Recipe HASH("ItemIronFrames")            # Recipe hash (see below)
define TargetCount 10                           # Craft when stock < this
define DispCraft HASH("AC-Status")              # Local status display
```

### Status Colors

| Color | Meaning |
|-------|---------|
| Green | Idle — stock at or above target |
| Orange | Actively crafting |
| Red | Error — missing materials, machine broken, etc. |

### How It Works

1. Each tick, reads `ExportCount` from the named machine (items produced since last `ClearMemory`)
2. If stock < `TargetCount` and machine is not in error:
   - Sets `RecipeHash` to the configured recipe
   - Writes `Activate 1` to trigger crafting
3. If machine reports `Error > 0`, enters error state (red display)
4. Writes status to `db.Setting` (0=idle, 1=crafting, 2=error) for craft-core

### Important Notes

- **One recipe per IC**: For multiple recipes on one machine, deploy multiple ICs, each with a different recipe hash
- **ExportCount tracks production**: Use `ClearMemory` on the machine to reset the counter when you collect items. The crafter will then start producing again toward `TargetCount`
- **Error detection**: If the machine lacks materials, it reports `Error > 0`. The crafter surfaces this as a red alert
- **Network-only**: Uses `lbn`/`sbn` — no direct pin wiring (d0) needed

---

## Craft Core

Lightweight aggregator that polls all `BOD-Craft` IC Housings for status.

### IC Housing Label

`BOD-CraftCore`

### Hardware

| Device | Label | Display Mode | Purpose |
|--------|-------|-------------|---------|
| IC Housing | BOD-CraftCore | -- | Runs craft-core.ic10 |
| LED Display (Small) | AC-Alert | 0 (number) | Master craft status (0/1/2) |
| LED Display (Small) | AC-Active | 0 (number) | Number of active crafters |
| LED Display (Small) | AC-Errors | 0 (number) | Error count (0 or 1) |

### Configuration

```
define DispAlert HASH("AC-Alert")     # Master status display
define DispActive HASH("AC-Active")   # Crafter count display
define DispErrors HASH("AC-Errors")   # Error display
```

### Display Colors

- **AC-Alert**: Green=all idle, Orange=some crafting, Red=errors
- **AC-Active**: Green=crafters exist, Orange=no crafters deployed
- **AC-Errors**: Green=no errors, Red=at least one machine errored

---

## Finding Recipe Hashes

Recipe hashes are game-specific values. Three ways to find them:

### Method 1: Stationpedia (Recommended)

1. Press **F1** in-game to open Stationpedia
2. Search for the item you want to craft
3. Note the **Recipe Hash** value
4. Use it in the `define Recipe` line

### Method 2: Configuration Cartridge

1. Insert a **Configuration Cartridge** into a Tablet
2. Point the Tablet at the crafting machine
3. Browse recipes — the hash is displayed for each

### Method 3: Community Wiki

Visit the Stationeers wiki **ItemHash** page for a comprehensive list of item and recipe hashes.

---

## Supported Machine Types

| Machine | PrefabHash for `MachineType` |
|---------|-----|
| Autolathe | `HASH("StructureAutolathe")` |
| Electronics Printer | `HASH("StructureElectronicsPrinter")` |
| Hydraulic Pipe Bender | `HASH("StructureHydraulicPipeBender")` |
| Tool Manufacturer | `HASH("StructureToolManufactory")` |
| Fabricator | `HASH("StructureFabricator")` |

Verify exact structure names in the in-game Stationpedia if a hash doesn't work.

---

## Setup Examples

### Example: Auto-Craft Iron Frames (Autolathe)

```
define MachineType HASH("StructureAutolathe")
define MachineName HASH("FrameCrafter")
define Recipe HASH("ItemIronFrames")
define TargetCount 20
define DispCraft HASH("AC-Frames")
```
Label your Autolathe as `FrameCrafter`. The IC will keep 20 iron frames in production.

### Example: Auto-Craft Cables (Electronics Printer)

```
define MachineType HASH("StructureElectronicsPrinter")
define MachineName HASH("CablePrinter")
define Recipe HASH("ItemCableCoil")
define TargetCount 50
define DispCraft HASH("AC-Cables")
```

### Example: Auto-Craft Pipes (Hydraulic Pipe Bender)

```
define MachineType HASH("StructureHydraulicPipeBender")
define MachineName HASH("PipeBender")
define Recipe HASH("ItemPipeIron")
define TargetCount 30
define DispCraft HASH("AC-Pipes")
```

---

## BOD Integration (Optional)

The craft-core writes its master alert to `db.Setting` on the `BOD-CraftCore` IC Housing. To include it in the BOD master alert, add this one line to `core/bod-core.ic10`:

In `PollMonitors` (needs a free register):
```
lbn rN HASH("StructureCircuitHousing") HASH("BOD-CraftCore") Setting Average
```

In `EvalMaster`:
```
max vMaster vMaster rN
```

**Note**: BOD Core is currently at 84/128 lines with r0-r14 used. r15 is available for this integration. This adds 2 lines (86/128).

If you don't need BOD integration, the auto-craft system works entirely standalone.

---

## Interface Contract

### Auto-Craft Template
- **Writes to `db.Setting`**: Status (0=idle, 1=crafting, 2=error)
- **Craft Core reads**: `lbn ... HASH("BOD-Craft") Setting Maximum`

### Craft Core
- **Writes to `db.Setting`**: Master craft alert (0=all idle, 1=crafting, 2=error)
- **BOD Core reads** (optional): `lbn ... HASH("BOD-CraftCore") Setting Average`

## Fail-Silent Behavior

If no auto-crafters are deployed, craft-core reads 0 from all `lbn` calls — displays show 0 crafters with green status. If craft-core itself isn't deployed, BOD Core (if integrated) reads 0 — no false alerts.
