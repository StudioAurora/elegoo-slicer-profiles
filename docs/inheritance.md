# ElegooSlicer Profile Inheritance

ElegooSlicer is based on OrcaSlicer and uses the same profile system. This document explains how it works.

## Core Concept: Delta Override

A profile is a JSON file. Each profile can declare a parent via the `"inherits"` field. The child profile only contains **settings that differ** from the parent. The slicer walks the entire chain and merges top-down — the most specific (last) value wins.

This means a user profile with 10 fields actually resolves to 300+ settings when you include everything inherited from its ancestors.

## Profile Types

| Type | ID Field | Directory | Example |
|------|----------|-----------|---------|
| `machine_model` | — | `machine/` | Printer hardware definition |
| `machine` | `printer_settings_id` | `machine/` | Printer variant (specific nozzle) |
| `filament` | `filament_settings_id` | `filament/` | Material settings |
| `process` | `print_settings_id` | `process/` | Print quality and speed settings |

## Storage Locations

| Location | `"from"` field | Description |
|----------|---------------|-------------|
| `system/Elegoo/` | `"system"` | Shipped by Elegoo, updated with slicer |
| `user/default/` | `"User"` | Your custom overrides |

macOS path: `~/Library/Application Support/ElegooSlicer/`

## Inheritance Chains

### Process (Elegoo Centauri Carbon 0.4mm nozzle)

```
fdm_process_common                              universal FDM defaults
  └── fdm_process_elegoo_common                 Elegoo-specific defaults
       └── fdm_process_elegoo_04020             0.4mm nozzle / 0.20mm layer
            └── 0.20mm Standard @Elegoo CC      system profile (setting_id: PECC04020)
                 ├── 0.16mm Optimal @Elegoo CC  system: overrides layer_height only
                 └── Your User Profile          user: only your changed settings
```

### Filament

```
fdm_filament_common                             universal filament defaults
  └── fdm_filament_tpu                          generic TPU defaults
       └── Elegoo TPU @base                     Elegoo TPU base
            └── Elegoo TPU 95A @ECC             system: printer-specific
```

### Machine

```
fdm_machine_common                              universal machine defaults
  └── Elegoo Centauri Carbon 0.4 nozzle         system profile
       └── Your User Override                   user: your changes
```

## Key Fields

| Field | Purpose |
|-------|---------|
| `"inherits"` | Parent profile name. Empty = root/standalone |
| `"from"` | `"system"` or `"User"` |
| `"instantiation"` | `"true"` = visible in UI, `"false"` = abstract parent only |
| `"compatible_printers"` | Restricts profile to specific printer variants |
| `"setting_id"` | Unique ID for sync |
| `"is_custom_defined"` | `"0"` = based on system, `"1"` = fully custom |

## Data Format

**Filament** values use single-element arrays:
```json
"nozzle_temperature": ["250"],
"fan_max_speed": ["30"]
```

**Process** values use plain strings:
```json
"outer_wall_speed": "50",
"layer_height": "0.16"
```

## The .info File

Each user override profile has a companion `.info` file (key=value, not JSON):

```
sync_info = create
user_id =
setting_id =
base_id = PECC04020
updated_time = 1779200622
```

`base_id` links to the system profile's `setting_id`.

## Resolution Order

When you select a profile:

1. Read the profile JSON
2. Follow `"inherits"` — search `user/` first, then `system/`
3. Recursively resolve parents until `"inherits"` is empty
4. Merge from root downward
5. Result: complete profile with all settings resolved

## Creating a New Profile

1. Pick a parent (usually a system profile)
2. Create a JSON file with only the fields you want to change
3. Set `"inherits"` to the parent name (exact match, case-sensitive)
4. Set `"name"` and the corresponding `_settings_id` to your profile name
5. Set `"from": "User"`
6. Create a matching `.info` file with the parent's `setting_id` as `base_id`
7. Place both files in `user/default/{type}/`
8. Restart the slicer
