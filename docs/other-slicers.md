# Porting the TPU Tuning to Other Slicers

The profiles in this repo are ElegooSlicer/OrcaSlicer JSON files. But the **tuning is not slicer-specific** — it's a set of physical values that any slicer can apply. This document lists every meaningful change as a slicer-agnostic setting, with the equivalent name in **Cura**, **PrusaSlicer**, and **Bambu Studio**, so you can reproduce these profiles anywhere.

> **Bambu Studio / OrcaSlicer** share almost all setting names with this repo — porting there is nearly 1:1. **Cura** and **PrusaSlicer** use different names and are the focus of the mapping tables below.

Values shown are for **Tinmorry TPU 95A on a 0.4 mm nozzle**. Treat them as a validated starting point, not gospel — every printer/filament combo needs a tweak or two.

---

## 1. The four settings that matter most

If you only copy four things, copy these. They are the difference between "TPU prints" and "TPU jams / under-extrudes."

| Setting | Value | Why |
|---------|-------|-----|
| **Print speed (walls)** | 20–35 mm/s | Flexible filament buckles in the extruder at PLA speeds → under-extrusion. This is the #1 factor. |
| **Max volumetric flow** | 3.2 mm³/s | Hard ceiling on how fast TPU can physically be pushed; caps every other speed automatically. |
| **Retraction distance** | 0.8 mm (direct drive) | Long retraction stretches the elastic filament and causes jams/gaps. Keep it short. |
| **Cooling** | Low base fan, boost on overhangs | Blanket 100 % fan ruins TPU layer/Z bonding. Low base, high only where overhangs need it. |

And the zeroth rule, before any setting: **dry your filament.** TPU is hygroscopic; wet TPU produces regular directional gaps in the first layer regardless of the profile.

---

## 2. Temperature

| Setting | This repo | Cura | PrusaSlicer | Notes |
|---------|-----------|------|-------------|-------|
| Nozzle temp | 225 °C | Printing Temperature | Nozzle → Other layers | Range 210–240 °C |
| First-layer nozzle | 230 °C | Printing Temperature Initial Layer | Nozzle → First layer | Slightly hotter for adhesion |
| Bed temp | 35 °C | Build Plate Temperature | Bed → Other layers | TPU self-adheres; hot bed = gooey/warped first layer |

---

## 3. Speed

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Outer wall | 25 mm/s | Outer Wall Speed | External perimeters |
| Inner wall | 35 mm/s | Inner Wall Speed | Perimeters |
| Sparse infill | 40 mm/s | Infill Speed | Infill |
| Solid/top infill | 40 / 25 mm/s | Top/Bottom Speed | Solid / Top solid infill |
| Bridges | 15 mm/s | — (see Bridge settings) | Bridges |
| First layer | 15 mm/s | Initial Layer Speed | First layer speed |
| Travel | 150 mm/s | Travel Speed | Travel |
| Gap fill | 25 mm/s | — | Gap fill |

**Max volumetric flow** (the real limiter): Cura = *Maximum Flow (Flow rate)* via the *Flow Rate Max* / plugin or material setting; PrusaSlicer = *Advanced → Max volumetric speed* (per-filament). Set to **3.2 mm³/s**.

---

## 4. Acceleration

Lower acceleration keeps flow steady and reduces ringing on flexible parts.

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Wall / default | 500 mm/s² | Wall Acceleration / Print Acceleration | External perimeters / default |
| First layer | 250 mm/s² | Initial Layer Acceleration | First layer |
| Travel | 1000 mm/s² | Travel Acceleration | Travel |

---

## 5. Retraction & Z-hop (filament-level)

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Retraction distance | 0.8 mm | Retraction Distance | Length (retraction) |
| Retraction speed | 25 mm/s | Retraction Speed | Retraction speed |
| Deretraction/prime speed | 15 mm/s | Retraction Prime Speed | Deretraction speed |
| Z-hop | 0.3 mm | Z Hop When Retracted | Lift Z |
| Z-hop style | Normal lift | Z Hop (standard) | Lift Z |

> Values are for a **direct-drive** extruder. On a Bowden setup, retraction distance will be considerably larger (3–6 mm) — direct-drive numbers do not transfer.

---

## 6. Cooling (the nuance most guides get wrong)

Don't just set fan to 100 %. Split it:

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Base fan (min) | 30 % | Fan Speed (Minimum) | Min fan speed |
| Base fan (max) | 50 % | Fan Speed (Maximum) | Max fan speed |
| Overhang fan boost | 100 % | Fan Speed Override / Overhang | Bridging fan speed (closest) |
| Overhang threshold | 50 % overhang | Fan overhang settings | — |
| Min layer time | 15 s | Minimum Layer Time | Slow down if layer time below |

Rationale: high fan improves overhang crispness but **weakens interlayer/Z bonding** on TPU. Keep the base low for strength, boost only on overhangs.

---

## 7. Walls, infill & first-layer reliability

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Wall count | 3 | Wall Line Count | Perimeters |
| Infill pattern | Gyroid | Infill Pattern → Gyroid | Fill pattern → Gyroid |
| Bridge flow | 0.95 | Bridge Flow | Bridge flow ratio |
| Brim width | 8 mm | Brim Width | Brim width |
| Elephant-foot comp. | 0.05 mm | Initial Layer Horizontal Expansion (negative) | Elephant foot compensation |

---

## 8. Overhang handling

OrcaSlicer/Bambu have graduated per-overhang speed controls and overhang-reverse that Cura/PrusaSlicer don't expose directly. Approximate them:

| This repo | What it does | Cura equivalent | PrusaSlicer equivalent |
|-----------|--------------|-----------------|------------------------|
| Overhang speeds 20 → 15 → 8 mm/s | Slows steeper overhangs progressively | Cura 5.x *Overhanging Wall Speed* (single value, use ~15) | *Overhangs → speed* (single value) |
| Extra perimeters on overhangs | Adds walls under overhangs for support | Extra Skin Wall Count (partial) | Extra perimeters if needed |
| Overhang reverse | Alternates direction to fight curling | — (not available) | — (not available) |

If your slicer lacks graduated overhang control, set a single conservative overhang wall speed of ~12–15 mm/s.

---

## 9. Support (for the "Support" variants)

TPU fuses to TPU, so the whole game is a **generous gap + thin interface** for clean removal.

| Setting | This repo | Cura | PrusaSlicer |
|---------|-----------|------|-------------|
| Support style | Snug | Support Placement → Everywhere + Tree/Normal | Snug / Support on build plate only = off |
| Z gap (top & bottom) | 0.2 mm (≈1 layer) | Support Z Distance | Contact Z distance |
| Interface layers | 1 top / 0 bottom | Support Interface (Enable, thin) | Interface layers |
| Interface spacing | 0.5 mm | Support Interface Line Distance (wide) | Interface pattern spacing |
| Support base spacing | 3 mm | Support Density (low) | Support spacing |
| XY distance | 0.35 mm | Support X/Y Distance | XY separation |
| Support on build plate only | **Off** | Support Placement → Everywhere | Uncheck "on build plate only" |
| Support/interface speed | 35 / 25 mm/s | Support Speed | Support speed |

> **Key for recesses/cavities:** *support on build plate only* must be **off**, otherwise a pocket that needs support from the model itself gets none.

---

## Quick sanity checklist when porting

1. Walls at 20–35 mm/s? ✔
2. Max volumetric flow capped at ~3.2 mm³/s? ✔
3. Retraction short (≤1 mm on direct drive)? ✔
4. Base fan low, overhang boost high — not blanket 100 %? ✔
5. Bed not too hot (≤40 °C)? ✔
6. Filament actually dried? ✔ (the one that bites everyone)
