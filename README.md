# ElegooSlicer Profiles

Custom print profiles for the **Elegoo Centauri Carbon** (CoreXY, direct drive), optimized for materials that need special process tuning beyond what the stock profiles provide.

## Profiles

### TPU Process Profiles

The stock ElegooSlicer installation includes TPU *filament* profiles but no dedicated *process* profile — meaning TPU prints run at the same 160-250 mm/s speeds as PLA/PETG, relying solely on volumetric flow limiting. These profiles fix that.

| Profile | Layer Height | Support | Use Case |
|---------|-------------|---------|----------|
| `0.20mm TPU @Elegoo CC 0.4 nozzle` | 0.20mm | off | General TPU printing — phone cases, gaskets, functional flex parts |
| `0.16mm TPU @Elegoo CC 0.4 nozzle` | 0.16mm | off | Fine detail TPU — cosmetic parts, thin-walled items |
| `0.20mm TPU Support @Elegoo CC 0.4 nozzle` | 0.20mm | on | TPU parts needing support (overhangs, recesses) |
| `0.16mm TPU Support @Elegoo CC 0.4 nozzle` | 0.16mm | on | Fine-detail TPU parts needing support |

**Key changes vs stock 0.20mm Standard:**

- Wall speeds: 25/35 mm/s (was 160/200)
- Infill: 40 mm/s (was 200)
- Acceleration: 500 mm/s² (was 5000)
- Travel: 150 mm/s (was 500)
- 3 walls, gyroid infill, wider brim
- Tuned, graduated overhang speeds (20 → 15 → 8 mm/s) + extra perimeters + `overhang_reverse`
- `reduce_infill_retraction` on (fewer retracts inside the print → fewer flex-filament jams)

The **Support** variants add a TPU-tuned support block: larger Z-gap, `snug` style, and a thin/sparse interface so support peels off cleanly (TPU fuses to TPU, so a generous gap is essential). `support_on_build_plate_only` is off so a recess can be supported from the model itself.

> **Note on inheritance:** all four profiles inherit **directly** from the system `0.20mm Standard @Elegoo CC 0.4 nozzle` — they are *not* chained to one another. ElegooSlicer silently refuses to load a user profile whose `inherits` points at another user profile that still inherits further (see [the gotcha below](#critical-gotcha-user-profiles-must-inherit-a-system-preset)). The 0.16mm profiles therefore repeat the shared TPU settings rather than inheriting them from the 0.20mm TPU profile.

### TPU Filament Profile

| Profile | Inherits | Material |
|---------|----------|----------|
| `Tinmorry TPU 95A @ECC` | `Elegoo TPU 95A @ECC` (system) | Tinmorry TPU 95A |

**Key changes vs the stock Elegoo TPU 95A base:**

- **Temps:** 225 °C nozzle / 230 °C first layer (validated against independent TPU 95A sources, not just the vendor sheet).
- **Retraction:** 0.8 mm @ 25 mm/s — short, to avoid stretching/jamming the elastic filament in the direct drive.
- **Z-hop:** 0.3 mm (Normal Lift) — stops the nozzle dragging over flexible prints.
- **Split cooling:** low base fan (30–50 %) for layer/Z strength, boosted to 100 % only on overhangs (`overhang_fan_threshold` 50 %). Blanket 100 % fan weakens TPU interlayer bonding.
- **`slow_down_layer_time` 15 s** — gives small parts time to cool between layers.

> **Moisture first.** TPU is hygroscopic. If a first layer shows regular, directional gaps (short bursts of under-extrusion) the cause is almost always **wet filament**, not the profile — dry it before chasing settings. Note that filament already in the PTFE tube stays wet even while the spool is in a dryer.

## Installation

### Profiles

Copy the profile files (both `.json` and `.info`) to your ElegooSlicer user directory:

```bash
# macOS
cp profiles/process/*.json profiles/process/*.info \
  ~/Library/Application\ Support/ElegooSlicer/user/default/process/

# Windows
copy profiles\process\*.json profiles\process\*.info \
  %APPDATA%\ElegooSlicer\user\default\process\
```

Restart ElegooSlicer. The profiles will appear in the process dropdown when the Elegoo Centauri Carbon 0.4 nozzle is selected.

### Claude Code Skill (optional)

If you use [Claude Code](https://claude.ai/claude-code) and want AI-assisted profile creation and debugging:

```bash
cp -r skill/SKILL.md ~/.claude/skills/elegoo-slicer-profiles/SKILL.md
```

## How Profiles Work

ElegooSlicer (based on OrcaSlicer) uses **JSON profiles with prototypal inheritance**. Each profile has an `"inherits"` field pointing to a parent. The child only stores settings that differ from the parent — everything else is resolved up the chain.

```
fdm_process_common
  └── fdm_process_elegoo_common
       └── fdm_process_elegoo_04020
            └── 0.20mm Standard @Elegoo CC 0.4 nozzle    ← system
                 ├── 0.20mm TPU @Elegoo CC 0.4 nozzle          ← this repo
                 ├── 0.16mm TPU @Elegoo CC 0.4 nozzle          ← this repo
                 ├── 0.20mm TPU Support @Elegoo CC 0.4 nozzle  ← this repo
                 └── 0.16mm TPU Support @Elegoo CC 0.4 nozzle  ← this repo
```

Every profile in this repo is a **direct, flat** child of the system preset — they deliberately do not inherit from each other.

### Critical gotcha: user profiles must inherit a system preset

ElegooSlicer only resolves **one hop** of user inheritance. A user process profile is loaded (and shown in the dropdown) only if its `inherits` points at a **system preset** or a standalone user root (`inherits: ""`). If it points at another *user* profile that itself still inherits further, the slicer **silently drops it** — no error, it just never appears in the UI. Adding `compatible_printers` does **not** fix it; flattening the inheritance to point straight at the system preset does.

That is why the 0.16mm and Support profiles here each repeat the shared TPU settings instead of chaining off `0.20mm TPU`. Trade-off: editing `0.20mm TPU` no longer propagates to the others — keep them in sync by hand.

See [docs/inheritance.md](docs/inheritance.md) for the full explanation.

## Using Another Slicer?

The tuning here is physical, not slicer-specific. [docs/other-slicers.md](docs/other-slicers.md) maps every change to its equivalent in **Cura**, **PrusaSlicer**, and **Bambu Studio** so you can reproduce these profiles anywhere.

## Compatibility

- **Printer**: Elegoo Centauri Carbon with 0.4mm nozzle
- **Slicer**: ElegooSlicer 1.3.x+ / OrcaSlicer 2.x+
- **Material**: Any TPU 95A (tested with Tinmorry TPU 95A, Elegoo TPU 95A)

## Contributing

Found better settings? Open a PR. Include:
1. What you changed and why
2. What material you tested with
3. Photos of results if possible

## License

MIT
