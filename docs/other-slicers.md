# Porting the TPU tuning to other slicers

The profiles in this repo are ElegooSlicer/OrcaSlicer JSON files, but the tuning itself has nothing to do with the slicer. It is a set of physical values, and any slicer can apply them. This page lists each change as a plain setting, with the equivalent name in Cura, PrusaSlicer, and Bambu Studio, so you can rebuild these profiles elsewhere.

Bambu Studio and OrcaSlicer use almost the same setting names as this repo, so porting there is close to a straight copy. Cura and PrusaSlicer name things differently, and those are what the tables below focus on.

The numbers are for Tinmorry TPU 95A on a 0.4 mm nozzle. They are a tested starting point, not a fixed rule. Every printer and filament combination needs a tweak or two.

---

## The four settings that matter most

If you copy nothing else, copy these four. They are the difference between a TPU print that works and one that jams or under-extrudes.

| Setting | Value | Why |
|---------|-------|-----|
| Print speed (walls) | 20-35 mm/s | Flexible filament buckles in the extruder at PLA speeds and starts under-extruding. This is the single biggest factor. |
| Max volumetric flow | 3.2 mm³/s | A hard ceiling on how fast TPU can physically move through the nozzle. It caps every other speed for you. |
| Retraction distance | 0.8 mm (direct drive) | Long retraction stretches the elastic filament and causes jams and gaps. Keep it short. |
| Cooling | Low base fan, boost on overhangs | A flat 100 % fan wrecks TPU layer and Z bonding. Run the base low and only spike it where overhangs need it. |

And the rule that comes before any setting: dry your filament. TPU pulls water out of the air. Wet TPU leaves regular, directional gaps in the first layer no matter how good the profile is.

---

## Temperature

| Setting | This repo | Cura | PrusaSlicer | Notes |
|---------|-----------|------|-------------|-------|
| Nozzle temp | 225 °C | Printing Temperature | Nozzle, other layers | Range 210-240 °C |
| First-layer nozzle | 230 °C | Printing Temperature Initial Layer | Nozzle, first layer | A little hotter to help it stick |
| Bed temp | 35 °C | Build Plate Temperature | Bed, other layers | TPU sticks to itself; a hot bed makes the first layer gooey and warped |

---

## Speed

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Outer wall | 25 mm/s | Outer Wall Speed | External perimeters |
| Inner wall | 35 mm/s | Inner Wall Speed | Perimeters |
| Sparse infill | 40 mm/s | Infill Speed | Infill |
| Solid/top infill | 40 / 25 mm/s | Top/Bottom Speed | Solid / Top solid infill |
| Bridges | 15 mm/s | Bridge settings | Bridges |
| First layer | 15 mm/s | Initial Layer Speed | First layer speed |
| Travel | 150 mm/s | Travel Speed | Travel |
| Gap fill | 25 mm/s | n/a | Gap fill |

The real limiter is max volumetric flow. In Cura it sits under the material settings (Maximum Flow); in PrusaSlicer it is under Advanced, Max volumetric speed, set per filament. Put it at 3.2 mm³/s and it quietly caps everything else.

---

## Acceleration

Lower acceleration keeps the flow steady and cuts ringing on flexible parts.

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Wall / default | 500 mm/s² | Wall Acceleration / Print Acceleration | External perimeters / default |
| First layer | 250 mm/s² | Initial Layer Acceleration | First layer |
| Travel | 1000 mm/s² | Travel Acceleration | Travel |

---

## Retraction and Z-hop (filament level)

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Retraction distance | 0.8 mm | Retraction Distance | Length (retraction) |
| Retraction speed | 25 mm/s | Retraction Speed | Retraction speed |
| Deretraction/prime speed | 15 mm/s | Retraction Prime Speed | Deretraction speed |
| Z-hop | 0.3 mm | Z Hop When Retracted | Lift Z |
| Z-hop style | Normal lift | Z Hop (standard) | Lift Z |

These numbers assume a direct-drive extruder. On a Bowden setup the retraction distance is much larger, usually 3 to 6 mm, so the direct-drive value does not carry over.

---

## Cooling

Setting the fan to a flat 100 % is the common mistake. Split it instead.

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Base fan (min) | 30 % | Fan Speed (Minimum) | Min fan speed |
| Base fan (max) | 50 % | Fan Speed (Maximum) | Max fan speed |
| Overhang fan boost | 100 % | Fan Speed Override / Overhang | Bridging fan speed (closest) |
| Overhang threshold | 50 % overhang | Fan overhang settings | n/a |
| Min layer time | 15 s | Minimum Layer Time | Slow down if layer time below |

A high fan makes overhangs crisper but weakens the layer and Z bonding on TPU. Keep the base low for strength and only push it up on overhangs.

---

## Walls, infill, and first-layer reliability

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Wall count | 3 | Wall Line Count | Perimeters |
| Infill pattern | Gyroid | Infill Pattern, Gyroid | Fill pattern, Gyroid |
| Bridge flow | 0.95 | Bridge Flow | Bridge flow ratio |
| Brim width | 8 mm | Brim Width | Brim width |
| Elephant-foot comp. | 0.05 mm | Initial Layer Horizontal Expansion (negative) | Elephant foot compensation |

---

## Overhang handling

OrcaSlicer and Bambu have per-overhang speed steps and an overhang-reverse option that Cura and PrusaSlicer do not expose directly. You can get close.

| This repo | What it does | Cura | PrusaSlicer |
|-----------|--------------|------|-------------|
| Overhang speeds 20, 15, 8 mm/s | Slows steeper overhangs progressively | Cura 5.x Overhanging Wall Speed (single value, use about 15) | Overhangs speed (single value) |
| Extra perimeters on overhangs | Adds walls under overhangs for support | Extra Skin Wall Count (partial) | Extra perimeters if needed |
| Overhang reverse | Alternates direction to fight curling | not available | not available |

If your slicer has no graduated overhang control, set one conservative overhang wall speed of about 12 to 15 mm/s.

---

## Support (for the Support variants)

TPU welds to TPU, so the whole thing comes down to a generous gap and a thin interface so the support peels off.

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Support style | Snug | Support Placement, Everywhere + Tree/Normal | Snug / Support on build plate only = off |
| Z gap (top and bottom) | 0.2 mm (about 1 layer) | Support Z Distance | Contact Z distance |
| Interface layers | 1 top / 0 bottom | Support Interface (enable, thin) | Interface layers |
| Interface spacing | 0.5 mm | Support Interface Line Distance (wide) | Interface pattern spacing |
| Support base spacing | 3 mm | Support Density (low) | Support spacing |
| XY distance | 0.35 mm | Support X/Y Distance | XY separation |
| Support on build plate only | Off | Support Placement, Everywhere | Uncheck "on build plate only" |
| Support/interface speed | 35 / 25 mm/s | Support Speed | Support speed |

For recesses and cavities, turn "support on build plate only" off. Otherwise a pocket that needs support from the model itself gets none.

---

## Quick sanity check when porting

1. Walls at 20-35 mm/s?
2. Max volumetric flow capped around 3.2 mm³/s?
3. Retraction short, under about 1 mm on direct drive?
4. Base fan low with an overhang boost, not a flat 100 %?
5. Bed no hotter than about 40 °C?
6. Filament actually dried? (the one everyone forgets)
