# Plan — CircleInlayRouterJig

**Status: DRAFT rev 2 (bearing-bit method), awaiting Bradley's approval. No CAD execution until it's approved (PROJECT_BOOTSTRAP Step 3).**

Rev 2, 2026-10-08: switched from a guide bushing to a **top-bearing pattern bit** (Bradley's
decision). Rev 1 (bushing kit) is in git history.

## Reference

`images/reference-IMG_0491.jpg`: the inlay is a laser-cut, laser-engraved wooden disc (a St. Benedict
medal, light plywood/basswood). It's engraved before inlaying, so the disc's cut edge is the only fit surface.
Size set: **Ø100 / 150 / 200 / 250**, with Ø150 first.

## Tooling

* **Router**: Milwaukee M18 FUEL compact router 2723-20 with its 5-3/4" (146.05 mm) round sub-base.
  No guide bushing is used.
* **Bit**: Jiiolioa top-bearing downcut flush-trim bit (Amazon listing): 3/8" cutter Ø, 3/8" cutting
  length, 1/4" shank, 2-1/8" overall. Above the cutter, in order: **two stacked bearings**, then a
  **lock collar with a set screw**. The bearing stack height isn't published (~5.2 mm scaled from the
  listing drawing, which is an estimate). **Measure all bit dims with calipers on arrival.**

## Concept

A printed plate with a circular through-hole. The bit's **bearing rides the hole wall**, and the cutter,
directly below the bearing, cuts the workpiece. The recess wall lands on the bearing's path.

Cross-section of the plate (workpiece at the bottom, z = 0 on the workpiece surface):

```
  z = TemplateThick ─┐   ← router base rides here; lock collar must stay ABOVE this line
                     │ bearing contact band (BearingBand)
  z = ReliefHeight  ─┤
                     │ underside relief: Ø ReliefDiameter, cutter spins here without touching plastic
  z = 0 ─────────────┘   ← workpiece surface; cutter goes RecessDepth below it
```

Vertical stack, with the bit plunged to full depth (cutter tip at −RecessDepth):

```
cutter:   −RecessDepth … CutterLength − RecessDepth
bearings: CutterLength − RecessDepth … CutterLength − RecessDepth + BearingStackHeight
collar:   above the bearings, so the plate's top face must stay below it
```

Diameters:

```
recess must be:  RecessDiameter       = InlayDiameter − LaserKerf + 2·InlayFitClearance
bit cuts at:     RecessDiameter       = TemplateHoleDiameter − BearingOD + CutterDiameter
therefore:       TemplateHoleDiameter = RecessDiameter + BearingOD − CutterDiameter
model hole:      TemplateHoleModelD   = TemplateHoleDiameter + 2·TemplateHolePrintComp
relief:          ReliefDiameter       = TemplateHoleDiameter + 2·ReliefClearance
```

`BearingOD − CutterDiameter` is nominally 0. It's kept explicit because a cheap bit's bearing
and cutter can differ by ~0.1 mm, which matters at this fit.

Heights:

```
ReliefHeight  = CutterLength − RecessDepth + ReliefGap                       (cutter top + gap)
TemplateThick = CutterLength − RecessDepth + BearingStackHeight − CollarGap  (top of bearings − gap)
BearingBand   = TemplateThick − ReliefHeight = BearingStackHeight − ReliefGap − CollarGap
```

The bearing band is independent of disc thickness: ~3.2 mm with the estimated stack.

Router-base support, with the bearing on the wall: the base center sits at `(Hole − BearingOD)/2`
from the template center. Its outer edge must stay on the plate:

```
TemplateMargin = RouterBaseDiameter/2 − BearingOD/2 + BaseSupportOverlap
TemplateSide   = TemplateHoleDiameter + 2·TemplateMargin
```

## PARAMETERS

All live in `Params.FCStd` → `VarSet`, created by `macros/00-bootstrap_params.FCMacro`.
**TBD** = placeholder until measured or confirmed.

| Name | Type | Default | Group | Meaning |
|---|---|---|---|---|
| `InlayDiameter` | Length | 150 mm | Inlay | Nominal laser-cut disc diameter (the size knob) |
| `LaserKerf` | Length | 0.15 mm **TBD** | Inlay | Diameter the disc loses to laser kerf (0 if the laser software compensates) |
| `RecessDepth` | Length | 3.0 mm **TBD** | Inlay | Disc thickness = routed depth. Drives plate height |
| `CutterDiameter` | Length | 9.525 mm **TBD** | Tooling | Measured cutter Ø (3/8") |
| `CutterLength` | Length | 9.525 mm **TBD** | Tooling | Measured cutting length (3/8") |
| `BearingOD` | Length | 9.525 mm **TBD** | Tooling | Measured bearing Ø |
| `BearingStackHeight` | Length | 5.2 mm **TBD** | Tooling | Both bearings together; an estimate from the drawing |
| `RouterBaseDiameter` | Length | 146.05 mm | Tooling | Milwaukee 2723-20 sub-base, 5-3/4" (sourced; measure to confirm) |
| `InlayFitClearance` | Length | 0.05 mm | Clearance | Per-side glue gap, disc ↔ recess wall |
| `TemplateHolePrintComp` | Length | 0.15 mm | Clearance | Per-side FDM "holes print small" offset. **Per material**, so tune after the test print |
| `ReliefClearance` | Length | 1.0 mm | Clearance | Per-side radial gap, cutter ↔ relief wall |
| `ReliefGap` | Length | 1.0 mm | Template | Vertical gap, cutter top ↔ relief ceiling |
| `CollarGap` | Length | 1.0 mm | Template | Vertical gap, plate top ↔ lock-collar bottom |
| `BaseSupportOverlap` | Length | 2 mm | Template | Plate beyond the router base's outer edge |
| `AlignNotchWidth` | Length | 3 mm | Template | Edge-midpoint alignment notch width |
| `AlignNotchDepth` | Length | 3 mm | Template | Notch depth into the plate edge |
| `RecessDiameter` | Length | *derived* | Derived | see Concept |
| `TemplateHoleDiameter` | Length | *derived* | Derived | see Concept |
| `TemplateHoleModelD` | Length | *derived* | Derived | Hole as modeled (includes print comp) |
| `ReliefDiameter` | Length | *derived* | Derived | see Concept |
| `ReliefHeight` | Length | *derived* | Derived | see Concept |
| `TemplateThick` | Length | *derived* | Derived | see Concept |
| `BearingBand` | Length | *derived* | Derived | Wall height the bearing actually rides (validation only; drives nothing) |
| `TemplateMargin` | Length | *derived* | Derived | see Concept |
| `TemplateSide` | Length | *derived* | Derived | see Concept |

Each concern gets its own knob: glue fit (`InlayFitClearance`), laser process (`LaserKerf`), FDM hole size
(`TemplateHolePrintComp`), cutter-to-plastic clearance (`ReliefClearance`, `ReliefGap`), and collar clearance
(`CollarGap`). Rev 1's `LeadInChamfer` is **dropped**. Any chamfer on the hole would eat into the
~3 mm bearing band, and the bit enters from above anyway.

Worked example (Ø150, placeholders): Recess **149.95**; Hole **149.95**; modeled hole **150.25**;
relief **Ø151.95 × 7.53** high; plate **290.48 × 290.48 × 10.73**; bearing band **3.2**; margin **70.26**.

## FEATURE TREE

`CircleInlayRouterJig.FCStd`, Body `RecessTemplate`. The plate's bottom face (workpiece side) is at z = 0,
centered on X/Y:

1. `DatumPlane_Base`: PartDesign::Plane on Origin XY (offset 0)
2. `Sk_Plate` on `DatumPlane_Base`: square, side `TemplateSide`, symmetric about the origin
3. `Pad_Plate`: Length = `TemplateThick` (+Z)
4. `Sk_Hole` on `DatumPlane_Base`: circle Ø `TemplateHoleModelD` at the origin
5. `Pocket_Hole`: ThroughAll
6. `Sk_Relief` on `DatumPlane_Base`: circle Ø `ReliefDiameter` at the origin
7. `Pocket_Relief`: Length = `ReliefHeight`, cutting up from the bottom face
8. `Sk_Notches` on `DatumPlane_Base`: 4 notch rectangles at the edge midpoints on the X/Y axes
9. `Pocket_Notches`: ThroughAll
10. `DatumPlane_Top`: offset `TemplateThick`. Created only if a top-face label is approved (Open Q 6)

No sketch attaches to a feature face.

**Print orientation: top face down** (flipped in the slicer). The relief then widens going up, so it
needs no supports. The bearing band prints on the first layers, so **enable elephant-foot compensation**.
Otherwise the bearing band, the one surface that sets the recess size, comes out undersize.

## CONSTRAINT STRATEGY

* Every sketch is fully constrained to the origin/axes using Symmetric + Horizontal/Vertical constraints. Every
  dimension is `setExpression`-bound to `<<Params>>#VarSet.<Name>`. Feature Lengths bind the same way.
* Sketches and features bind to the **derived** names, so each formula lives once, in the VarSet.
* Built with typed MCP tools (`create_sketch`, `add_sketch_*`, `sketcher_add_constraint_*`, `pad_sketch`,
  `pocket_sketch`, `create_datum_plane`) and recorded as `macros/NN-*.FCMacro`.

## VALIDATION

1. `python3 scripts/audit_parametric.py` is clean.
2. **Flex tests** (a binding alone isn't proof the Param drives geometry):
   - `InlayDiameter` 150 → 100: the measured hole edge radius = `TemplateHoleModelD/2`, and the plate bbox = `TemplateSide`.
   - `RecessDepth` 3 → 4: the plate gets 1 mm thinner and the relief 1 mm shorter; the bearing band stays the same.
   Restore the values afterwards.
3. `validate_object` / `part_check_shape`: a valid closed solid, centered on X/Y.
4. Geometry asserts: `TemplateSide ≤ 350` (K2 Plus bed); `BearingBand ≥ 2.5`; `ReliefHeight < TemplateThick`.
5. **Bench checks before the first rout:**
   - Measure the bit (cutter Ø/length, bearing Ø, bearing stack height) and update the Params.
   - Collet check: with the bit set ~`TemplateThick + RecessDepth` (~13.7 mm) below the base, confirm
     ≥ 3/4" of shank is in the collet and the collet nut clears the lock collar.
6. **Physical:** print **Ø100** first, in the production material and top face down. Measure the
   bearing band Ø on two axes. Set the slicer's XY shrink from the scale error, then `TemplateHolePrintComp`
   from any remaining constant offset. Check the plate's flatness. Rout scrap in **one full-depth pass**
   and dry-fit a laser-cut disc. Tune `InlayFitClearance` / `LaserKerf`.

## USAGE CONSTRAINT: ONE FULL-DEPTH PASS

The plate's height is computed for the cutter at full `RecessDepth`. On a shallower pass the cutter sits
higher and **cuts into the plate above the relief**. For the 2–4 mm plywood discs, a 3/8" downcut at full
depth is a normal single pass for a compact router. If multiple passes are ever needed, raise `ReliefGap`
by the shallowest pass's shortfall. That shrinks `BearingBand`, so check the ≥ 2.5 mm assert.

## MATERIAL

Candidates: **PA6-CF12** (brand TBD) or **Polymaker Fiberon PETG-rCF08**. **PETG-rCF08 is recommended**,
because PA6 absorbs moisture and swells, so a nylon hole drifts with shop humidity.

* Shrink is a percentage and scales with size: ~0.5 mm diametral on a 200 mm hole at 0.25%. **Compensate
  it in the slicer's per-filament XY shrinkage**, not in CAD. One true-size STL then serves both materials.
  *(Assumes Creality Print/Orca has a per-filament shrinkage setting. Verify it.)*
* CF filament is abrasive, so it needs a **hardened nozzle**. Dry before printing (PETG-rCF08: 65 °C / 3 h, per the Polymaker TDS).
* Flatness matters, since warp tilts the router. Print in the enclosed chamber and check with a straightedge.
* The bearings ride on printed CF-PETG; expect slow wear of the bearing band over many uses.

## SIZE SET & BED FIT

Every size comes from one model by changing `InlayDiameter`. `macros/NN-export_all_sizes.FCMacro` will loop
through the set, recompute, export `stl/RecessTemplate-D<size>.stl`, and restore the original value.

K2 Plus 350 mm bed (placeholders; margin 70.26, hole ≈ disc − 0.05):

| Disc | Plate side | Fit |
|---|---|---|
| 100 | 240.5 | ✅ 109 mm spare |
| 150 | 290.5 | ✅ 59 mm spare |
| 200 | 340.5 | ✅ 9.5 mm spare. Confirm the real printable area |
| 250 | 390.5 | ❌ 40.5 mm over (Open Q 4) |

## OPEN QUESTIONS (need answers before approval)

1. **Disc thickness**: sets `RecessDepth` and therefore plate height. Is it the same for every size?
2. **Bit measurements** on arrival: cutter Ø, cutting length, bearing Ø, bearing stack height. Do the collet check (Validation 5).
3. **Laser kerf**: does the laser software compensate? If not, what's the measured kerf?
4. **Ø250 strategy**: (a) a **segmented plate**, 2 or 4 keyed pieces with a `SegmentCount` Param and
   its own joint-clearance Param, recommended. The bearing crosses the seams, so they must register flush.
   (b) A thinner margin on Ø250 only, leaving the base overhanging the plate by ~20 mm. (c) A narrow ring
   plus a separate sub-base to steady the router.
5. **Workholding**: double-sided tape only, or countersunk screw/clamp holes?
6. **Size label** engraved on the top face (needs `DatumPlane_Top`)?
7. **Plate outline**: solid square (simplest) or round (less plastic, notches stay on the axes)?
8. **PA6-CF12 brand**, only if it's still a candidate.
