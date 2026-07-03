# ElegooSlicer Profile Inheritance

ElegooSlicer is based on OrcaSlicer and uses the same profile system. This document explains how it works.

## Core Concept: Delta Override

A profile is a JSON file. Each profile can declare a parent via the `"inherits"` field. The child profile only contains **settings that differ** from the parent. The slicer walks the entire chain and merges top-down, and the most specific (last) value wins.

This means a user profile with 10 fields actually resolves to ~140-150 settings when you include everything inherited from its ancestors.

## Profile Types

| Type | ID Field | Directory | Example |
|------|----------|-----------|---------|
| `machine_model` | n/a | `machine/` | Printer hardware definition |
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

> Every user profile hangs directly off a system preset. The loader depends on this, as explained below.

## Critical Constraint: user profiles must inherit a system preset

ElegooSlicer resolves only **one hop** of *user* inheritance. A user profile is loaded, and shown in the UI dropdown, only when its `inherits` points at either:

- a **system preset**, or
- a **standalone user root** (`inherits: ""`).

If a user profile's `inherits` points at **another user profile that itself still inherits further**, the slicer **silently drops it**. There is no error and no warning; the profile simply never appears in the dropdown.

```
0.20mm Standard @Elegoo CC   (system)
  └── 0.20mm TPU  (user)          loads    (inherits a system preset)
       └── 0.16mm TPU  (user)     hidden   (inherits a user delta that still inherits)
```

Adding an explicit `compatible_printers` list does not fix it. That was the first hypothesis, and it made no difference.

To fix it, flatten the profile. Rewrite the hidden profile so its `inherits` points straight at the system preset, and merge every inherited delta value into the file itself.

```
0.20mm Standard @Elegoo CC   (system)
  ├── 0.20mm TPU  (user)      loads
  └── 0.16mm TPU  (user)      loads, now inherits the SYSTEM preset, with 0.20mm TPU's
                              deltas copied in verbatim
```

The cost is that the profiles are now self-contained. Editing `0.20mm TPU` no longer propagates to `0.16mm TPU` or the Support variants, so the shared settings have to be kept in sync by hand.

### How to check whether a profile even loaded

If a profile is missing from the dropdown, tabulate every user profile's `inherits` value against what *is* visible. In practice the visible ones all inherit a system preset (or a standalone root); the hidden ones all inherit a still-inheriting user delta. That pattern is the tell.

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
| `"instantiation"` | System profiles only. `"true"` = visible in UI, `"false"` = abstract parent |
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

`base_id` links to the root system ancestor's `setting_id` (not necessarily the immediate parent).

## Resolution Order

When you select a profile:

1. Read the profile JSON
2. Follow `"inherits"` and resolve against available profiles (user overrides can shadow system names)
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
