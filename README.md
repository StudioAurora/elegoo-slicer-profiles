# ElegooSlicer Profiles

Custom print profiles for the **Elegoo Centauri Carbon** (CoreXY, direct drive), optimized for materials that need special process tuning beyond what the stock profiles provide.

## Profiles

### TPU Process Profiles

The stock ElegooSlicer installation includes TPU *filament* profiles but no dedicated *process* profile — meaning TPU prints run at the same 160-250 mm/s speeds as PLA/PETG, relying solely on volumetric flow limiting. These profiles fix that.

| Profile | Layer Height | Use Case |
|---------|-------------|----------|
| `0.20mm TPU @Elegoo CC 0.4 nozzle` | 0.20mm | General TPU printing — phone cases, gaskets, functional flex parts |
| `0.16mm TPU @Elegoo CC 0.4 nozzle` | 0.16mm | Fine detail TPU — cosmetic parts, thin-walled items |

**Key changes vs stock 0.20mm Standard:**

- Wall speeds: 25/35 mm/s (was 160/200)
- Infill: 40 mm/s (was 200)
- Acceleration: 500 mm/s² (was 5000)
- Travel: 120 mm/s (was 500)
- 3 walls, gyroid infill, wider brim
- Tuned overhang and bridge speeds

The 0.16mm profile inherits from the 0.20mm TPU profile and further reduces speeds for finer detail work.

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
                 └── 0.20mm TPU @Elegoo CC 0.4 nozzle    ← this repo
                      └── 0.16mm TPU @Elegoo CC 0.4 nozzle
```

See [docs/inheritance.md](docs/inheritance.md) for the full explanation.

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
