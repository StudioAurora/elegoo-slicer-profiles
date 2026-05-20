---
name: elegoo-slicer-profiles
description: Manage ElegooSlicer/OrcaSlicer 3D printer profiles — create, edit, debug, and validate presets for filament, process, and machine configurations. Covers the full inheritance system, delta-override mechanics, standalone vs override profiles, .info metadata, and profile validation. Use when working with slicer profiles, creating new material/print presets, debugging inheritance chains, or bulk-editing profile settings.
---

# ElegooSlicer / OrcaSlicer Profile Management

ElegooSlicer is based on OrcaSlicer. Both use JSON-based profiles with a prototypal inheritance system. This skill covers the full profile lifecycle.

## Paths

| Location | Purpose |
|----------|---------|
| `~/Library/Application Support/ElegooSlicer/system/Elegoo/` | System profiles (read-only, updated by slicer) |
| `~/Library/Application Support/ElegooSlicer/user/default/` | User profiles (your overrides) |
| `~/Projects/OrcaSlicer.wiki/` | Official documentation |

Each location has three subdirectories: `filament/`, `process/`, `machine/`.

## Profile Types

| Type | JSON field | Stored in | Example |
|------|-----------|-----------|---------|
| `machine_model` | — | `machine/` | `Elegoo Centauri Carbon.json` |
| `machine` | `printer_settings_id` | `machine/` | `Elegoo Centauri Carbon 0.4 nozzle.json` |
| `filament` | `filament_settings_id` | `filament/` | `Elegoo PETG PRO @ECC.json` |
| `process` | `print_settings_id` | `process/` | `0.16mm Optimal @Elegoo CC 0.4 nozzle.json` |

## Inheritance System

### Core Mechanic: Delta Override

A child profile contains **only the settings that differ** from its parent. The slicer walks the `"inherits"` chain and merges top-down — last (most specific) wins.

### Key Fields

| Field | Purpose |
|-------|---------|
| `"inherits"` | Parent profile name. Empty string = root/standalone. |
| `"from"` | `"system"` (Elegoo-shipped) or `"User"` (custom) |
| `"instantiation"` | `"true"` = visible in UI, `"false"` = abstract parent only |
| `"compatible_printers"` | Array of printer variant names this profile works with |
| `"setting_id"` | Unique ID for system sync |
| `"is_custom_defined"` | `"0"` = based on system profile, `"1"` = fully custom |
| `"type"` | `"filament"`, `"process"`, `"machine"`, or `"machine_model"` |

### Inheritance Chains (Elegoo Centauri Carbon)

#### Process chain

```
fdm_process_common
  └─ fdm_process_elegoo_common
       └─ fdm_process_elegoo_04020          (0.4mm nozzle / 0.20mm layer)
            └─ 0.20mm Standard @Elegoo CC 0.4 nozzle   [system, setting_id: PECC04020]
                 └─ 0.16mm Optimal @Elegoo CC 0.4 nozzle   [system, overrides: layer_height]
                      └─ Your User Profile              [user, overrides: speeds, walls, etc.]
```

#### Filament chain

```
fdm_filament_pet
  └─ Elegoo PETG @base                    [filament_id: EPETGB00]
       └─ Elegoo PETG PRO @ECC            [system, setting_id: EPETGPROECC]
            └─ Your User Override          [user, only changed fields]
```

#### Machine chain

```
fdm_machine_common
  └─ Elegoo Centauri Carbon 0.4 nozzle    [system, setting_id: ECC04]
       └─ Your User Override              [user, e.g. multi bed types]
```

### Resolution Order

When the slicer loads a profile:

1. Read the user profile JSON
2. Follow `"inherits"` — search `user/` first, then `system/`
3. Recursively resolve parents until `"inherits"` is empty
4. Merge from root downward — each level overrides parent values
5. Result: complete profile with all ~300 settings resolved

## Profile Anatomy

### Override Profile (typical user profile)

Minimal — contains only deltas from parent:

```json
{
    "filament_settings_id": ["Elegoo PETG PRO @ECC - No Fan"],
    "from": "User",
    "inherits": "Elegoo PETG PRO @ECC",
    "is_custom_defined": "0",
    "name": "Elegoo PETG PRO @ECC - No Fan",
    "complete_print_exhaust_fan_speed": ["0"],
    "during_print_exhaust_fan_speed": ["0"],
    "version": "1.2.0.19"
}
```

Requires a companion `.info` file with `base_id` linking to the system profile.

### Standalone Profile (no inheritance)

Contains ALL settings — fully self-contained:

```json
{
    "type": "process",
    "name": "0.16mm Optimal PETG Classic Standalone",
    "from": "User",
    "inherits": "",
    "is_custom_defined": "0",
    "layer_height": "0.16",
    "outer_wall_speed": "50",
    ... (300+ fields)
}
```

No `.info` file needed. Stored in `base/` subdirectory by convention.

### The `.info` File

Key-value metadata (not JSON):

```
sync_info = update
user_id =
setting_id =
base_id = EPETGPROECC
updated_time = 1774382284
```

| Field | Purpose |
|-------|---------|
| `sync_info` | `create` or `update` — sync status |
| `base_id` | Links to system profile's `setting_id` |
| `updated_time` | Unix timestamp of last modification |

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Filament | `Brand Material @PrinterAbbrev` | `Elegoo PETG PRO @ECC` |
| Process | `LayerHeight Quality @Printer Nozzle` | `0.16mm Optimal @Elegoo CC 0.4 nozzle` |
| Machine variant | `Vendor Printer Nozzle` | `Elegoo Centauri Carbon 0.4 nozzle` |
| User override | `Parent Name - Variant` | `Elegoo PETG PRO @ECC - No Fan` |

The `@` separates the material/quality name from the printer it targets.

## Creating a New Profile

### Override Profile (recommended)

1. Choose a parent profile (system or user)
2. Create JSON with only the fields you want to change
3. Create matching `.info` file
4. Place both in the correct `user/default/{type}/` directory

#### Step-by-step: New Filament Override

```bash
# 1. Identify parent's setting_id from system profile
cat "system/Elegoo/filament/ECC/Elegoo PETG PRO @ECC.json" | grep setting_id
# → "setting_id": "EPETGPROECC"
```

```json
// 2. Create: user/default/filament/My PETG Variant @ECC.json
{
    "filament_settings_id": ["My PETG Variant @ECC"],
    "from": "User",
    "inherits": "Elegoo PETG PRO @ECC",
    "is_custom_defined": "0",
    "name": "My PETG Variant @ECC",
    "nozzle_temperature": ["245"],
    "fan_max_speed": ["50"],
    "version": "1.3.2.9"
}
```

```ini
; 3. Create: user/default/filament/My PETG Variant @ECC.info
sync_info = create
user_id =
setting_id =
base_id = EPETGPROECC
updated_time = 1716100000
```

#### Step-by-step: New Process Override

```json
// user/default/process/0.16mm My Custom @Elegoo CC 0.4 nozzle.json
{
    "from": "User",
    "inherits": "0.16mm Optimal @Elegoo CC 0.4 nozzle",
    "is_custom_defined": "0",
    "name": "0.16mm My Custom @Elegoo CC 0.4 nozzle",
    "print_settings_id": "0.16mm My Custom @Elegoo CC 0.4 nozzle",
    "outer_wall_speed": "40",
    "inner_wall_speed": "80",
    "sparse_infill_density": "20%",
    "version": "1.3.2.9"
}
```

### Standalone Profile

Use when you want full control with no dependency on system profiles:

1. Export or copy a fully resolved profile
2. Set `"inherits": ""`
3. Place in `user/default/{type}/base/` by convention
4. No `.info` file needed

### Machine Override

```json
// user/default/machine/Elegoo Centauri Carbon 0.4 nozzle - my variant.json
{
    "from": "User",
    "inherits": "Elegoo Centauri Carbon 0.4 nozzle",
    "is_custom_defined": "0",
    "name": "Elegoo Centauri Carbon 0.4 nozzle - my variant",
    "printer_settings_id": "Elegoo Centauri Carbon 0.4 nozzle - my variant",
    "support_multi_bed_types": "1",
    "version": "1.3.2.9"
}
```

## Data Format Rules

### Filament settings use arrays

Filament values are wrapped in single-element arrays:

```json
"nozzle_temperature": ["250"],
"fan_max_speed": ["30"],
"filament_density": ["1.04"]
```

### Process settings use plain strings

Process values are plain strings:

```json
"outer_wall_speed": "50",
"layer_height": "0.16",
"sparse_infill_density": "15%"
```

### Percentage values

Some fields use percentage strings: `"15%"`, `"50%"`, `"100%"`.

### Multi-value arrays (rare)

`compatible_printers` uses actual multi-element arrays:

```json
"compatible_printers": [
    "Elegoo Centauri Carbon 0.4 nozzle",
    "Elegoo Centauri Carbon 0.6 nozzle"
]
```

## Debugging Profiles

### Problem: Settings not taking effect

1. Check the inheritance chain — a parent may override your value
2. Verify `"inherits"` name matches exactly (case-sensitive)
3. Check `"compatible_printers"` — profile may be filtered out for your printer
4. Confirm the field name is correct (check system profile for valid keys)

### Problem: Profile not showing in UI

1. Verify `"instantiation"` is `"true"` (or absent — defaults to true for user profiles)
2. Check `"compatible_printers"` includes your active printer variant
3. Verify JSON is valid (no trailing commas, proper quoting)
4. Restart the slicer — profiles are cached

### Problem: Profile shows wrong values

Trace the full inheritance chain:

```bash
# Find what a user profile inherits
grep '"inherits"' "user/default/process/MyProfile.json"

# Find the system parent
find "system/" -name "ParentName.json" -exec grep '"inherits"' {} \;

# Continue up the chain until inherits is empty
```

### Comparing resolved values

To see the effective difference between two profiles, compare their full resolved chains. For override profiles, the JSON only shows deltas — the actual active value may come from any ancestor.

## Bulk Operations

### Find all profiles inheriting from a specific parent

```bash
grep -rl '"inherits": "Elegoo PETG PRO @ECC"' user/default/filament/
```

### Find all profiles with a specific setting

```bash
grep -rl '"fan_max_speed"' user/default/filament/
```

### List all inheritance roots (standalone profiles)

```bash
grep -rl '"inherits": ""' user/default/
```

### Get all user profile names

```bash
find user/default -name "*.json" -exec grep -l '"from": "User"' {} \; | \
  xargs -I{} grep '"name"' {} | sort
```

### Update a setting across multiple profiles

```bash
# Example: change version in all user filament profiles
find user/default/filament -name "*.json" -exec \
  sed -i '' 's/"version": "1.2.0.19"/"version": "1.3.2.9"/' {} \;
```

## Version Field

The `"version"` field should match the slicer version the profile was created with or last edited in. Current versions seen:

- `1.2.0.19` — older profiles
- `1.3.0.11` — mid-range
- `1.3.2.9` — recent
- `2.2.0` / `2.2.0.0` — standalone profiles (OrcaSlicer format)

When creating new profiles, use the version of the slicer you're currently running.

## Validation Checklist

Before using a new profile:

- [ ] JSON is valid (no syntax errors)
- [ ] `"name"` matches filename (without `.json`)
- [ ] `"inherits"` references an existing profile name
- [ ] `"{type}_settings_id"` matches `"name"`
- [ ] `"from"` is set to `"User"`
- [ ] `.info` file exists (for override profiles)
- [ ] `.info` `base_id` matches parent's `setting_id`
- [ ] Field names match valid slicer settings
- [ ] Data format correct (arrays for filament, strings for process)
- [ ] `"compatible_printers"` is appropriate (or inherited)
- [ ] `updated_time` in `.info` is a valid Unix timestamp

## Reference: Common Settings

### Filament (frequently overridden)

| Setting | Type | Example | Purpose |
|---------|------|---------|---------|
| `nozzle_temperature` | array | `["250"]` | Hotend temp |
| `nozzle_temperature_initial_layer` | array | `["255"]` | First layer temp |
| `cool_plate_temp` | array | `["60"]` | Bed temp (cool plate) |
| `eng_plate_temp` | array | `["100"]` | Bed temp (engineering plate) |
| `fan_max_speed` | array | `["30"]` | Part cooling fan max % |
| `fan_min_speed` | array | `["10"]` | Part cooling fan min % |
| `filament_flow_ratio` | array | `["0.98"]` | Flow multiplier |
| `filament_max_volumetric_speed` | array | `["15"]` | Max extrusion rate mm3/s |
| `pressure_advance` | array | `["0.024"]` | Linear advance value |
| `filament_retraction_length` | array | `["0.8"]` | Retraction distance mm |
| `filament_retraction_speed` | array | `["35"]` | Retraction speed mm/s |
| `close_fan_the_first_x_layers` | array | `["3"]` | No-fan initial layers |
| `filament_type` | array | `["PETG"]` | Material type identifier |
| `filament_density` | array | `["1.24"]` | g/cm3 for cost calc |

### Process (frequently overridden)

| Setting | Type | Example | Purpose |
|---------|------|---------|---------|
| `layer_height` | string | `"0.16"` | Layer height mm |
| `outer_wall_speed` | string | `"50"` | Outer perimeter speed mm/s |
| `inner_wall_speed` | string | `"100"` | Inner perimeter speed mm/s |
| `sparse_infill_speed` | string | `"100"` | Infill speed mm/s |
| `sparse_infill_density` | string | `"15%"` | Infill percentage |
| `wall_loops` | string | `"2"` | Number of perimeters |
| `top_shell_layers` | string | `"5"` | Top solid layers |
| `bottom_shell_layers` | string | `"3"` | Bottom solid layers |
| `top_surface_speed` | string | `"50"` | Top surface speed mm/s |
| `wall_generator` | string | `"classic"` | `classic` or `arachne` |
| `wall_sequence` | string | `"inner wall/outer wall"` | Print order |
| `enable_support` | string | `"0"` | Support on/off |
| `support_type` | string | `"tree(auto)"` | Support style |
| `print_flow_ratio` | string | `"0.95"` | Global flow multiplier |

## Documentation Reference

Full OrcaSlicer documentation at `~/Projects/OrcaSlicer.wiki/`:

| Path | Content |
|------|---------|
| `developer_reference/how_to_create_profiles.md` | Official profile creation guide |
| `developer_reference/preset_and_bundle.md` | Preset/PresetBundle/PresetCollection classes |
| `developer_reference/slicing_hierarchy.md` | Slicing call hierarchy |
| `material_settings/dependencies/` | Compatible printers/processes conditions |
| `material_settings/setting overrides/` | Material-level setting overrides |
| `print_settings/` | All process setting documentation |
| `printer_settings/` | Machine setting documentation |
