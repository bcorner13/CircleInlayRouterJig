# Plan — CircleInlayRouterJig

**Status: DRAFT — awaiting Bradley's approval. No CAD execution until approved (PROJECT_BOOTSTRAP Step 3).**

## Reference

`images/reference-IMG_0491.jpg`: the inlay is a laser-cut, laser-engraved wooden disc
(St. Benedict medal, light plywood/basswood). The engraving is done before inlaying, so the
disc's cut edge is the only fit surface. Its scale can't be read from the photo, but it's
150 mm in diameter (confirmed by Bradley 2026-10-08). That makes **bed fit and router-base support**
real constraints (see Validation step 4 and Open Questions).

## Concept

One printed plate with a circular through-hole. The router's guide bushing rides the
inside of the hole, and the bit cuts a recess **larger** than the bushing path. The
inlay disc is laser cut, so there is no plug template. The template only has to
produce a recess that fits a disc of known diameter.

Geometry (all diameters):

```
bit cuts at:     RecessDiameter       = TemplateHoleDiameter - BushingOD + BitDiameter
recess must be:  RecessDiameter       = InlayDiameter - LaserKerf + 2*InlayFitClearance
therefore:       TemplateHoleDiameter = RecessDiameter + BushingOD - BitDiameter
model hole:      TemplateHoleModelD   = TemplateHoleDiameter + 2*TemplateHolePrintComp
```

`LaserKerf` is the diameter the disc loses when the laser cuts on the nominal line.
Set it to 0 if the laser software already compensates for kerf.

## PARAMETERS

All live in `Params.FCStd` → `VarSet`, created by `macros/00-bootstrap_params.FCMacro`.
Values marked **TBD** are placeholders until Bradley gives the real hardware.

| Name | Type | Default | Group | Meaning |
|---|---|---|---|---|
| `InlayDiameter` | Length | 150 mm | Inlay | Nominal laser-cut disc diameter (the size knob) |
| `LaserKerf` | Length | 0.15 mm **TBD** | Inlay | Diameter lost to laser kerf (0 if compensated in the laser software) |
| `InlayFitClearance` | Length | 0.05 mm | Clearance | Per-side glue gap, disc ↔ recess wall |
| `BushingOD` | Length | 15.875 mm **TBD** | Tooling | Porter-Cable-style guide bushing OD (5/8" assumed) |
| `BushingProtrusion` | Length | 6.0 mm **TBD** | Tooling | How far the bushing sticks out below the router base |
| `BitDiameter` | Length | 3.175 mm **TBD** | Tooling | Router bit cutting diameter (1/8"; the router takes 1/4" shanks) |
| `RouterBaseDiameter` | Length | 146.05 mm | Tooling | Milwaukee M18 FUEL 2723-20 template sub-base, 5-3/4" (sourced; measure to confirm) |
| `TemplateHolePrintComp` | Length | 0.15 mm | Clearance | Per-side FDM compensation, since printed holes come out undersize. Tune after test print |
| `BushingFloorGap` | Length | 1.5 mm | Template | Clearance between bushing tip and workpiece |
| `BaseSupportOverlap` | Length | 2 mm | Template | Extra plate beyond the router base's outer edge, with the bushing touching the hole wall |
| `LeadInChamfer` | Length | 0.8 mm | Template | Top-edge chamfer on the hole so the bushing drops in cleanly |
| `AlignNotchWidth` | Length | 3 mm | Template | V/slot notch at each edge midpoint, used to align to layout lines |
| `AlignNotchDepth` | Length | 3 mm | Template | Notch depth into plate edge |
| `RecessDiameter` | Length | *derived* | Derived | Expression, see Concept |
| `TemplateHoleDiameter` | Length | *derived* | Derived | Expression, see Concept |
| `TemplateHoleModelD` | Length | *derived* | Derived | Hole as modeled (includes print comp) |
| `TemplateMargin` | Length | *derived* | Derived | `RouterBaseDiameter/2 − BushingOD/2 + BaseSupportOverlap` (= 67.09) |
| `TemplateSide` | Length | *derived* | Derived | `TemplateHoleDiameter + 2*TemplateMargin` |
| `TemplateThick` | Length | *derived* | Derived | `BushingProtrusion + BushingFloorGap` (the bushing can't bottom out on the workpiece) |

Clearance concepts are decoupled: glue fit (`InlayFitClearance`), laser process
(`LaserKerf`), and FDM print fit (`TemplateHolePrintComp`) each get their own knob.

Worked example (Ø150 disc, tooling placeholders): Recess = 150 − 0.15 + 0.10 = **149.95**;
Hole = 149.95 + 15.875 − 3.175 = **162.65**; modeled hole **162.95**; margin **67.09**; plate
**296.82 × 296.82 × 7.5**.

Why the margin is base radius − bushing radius: with the bushing touching the hole wall, the
router base's outer edge sits at `(Hole − BushingOD)/2 + RouterBaseDiameter/2` from center. The
plate edge must reach that point, at `Hole/2 + TemplateMargin`. That's the worst case, along the
X/Y axes; the square's corners give extra support on the diagonals.

## FEATURE TREE

`CircleInlayRouterJig.FCStd`, Body `RecessTemplate`, centered at (0,0,0):

1. `DatumPlane_Base`: PartDesign::Plane on Origin XY (offset 0)
2. `Sk_Plate` on `DatumPlane_Base`: square, `TemplateSide`, symmetric about the origin
3. `Pad_Plate`: Length = `TemplateThick`
4. `DatumPlane_Top`: offset `TemplateThick` from XY (for any top-face sketches)
5. `Sk_Hole` on `DatumPlane_Base`: circle, diameter = `TemplateHoleModelD`, center on origin
6. `Pocket_Hole`: ThroughAll (reversed as needed)
7. `Sk_Notches` on `DatumPlane_Base`: 4 notch rectangles at edge midpoints, on the X/Y axes
8. `Pocket_Notches`: ThroughAll
9. `Chamfer_LeadIn`: top circular edge of the hole, Size = `LeadInChamfer`

No sketch attaches to a feature face. Both base and top sketches use datum planes.

## CONSTRAINT STRATEGY

* Every sketch fully constrained to the origin/axes with Symmetric + Horizontal/Vertical
  geometric constraints. Every dimension is `setExpression`-bound to
  `<<Params>>#VarSet.<Name>`.
* Built via typed MCP tools (`create_sketch`, `add_sketch_*`, `sketcher_add_constraint_*`,
  `pad_sketch`, `pocket_sketch`, `chamfer_edges`) and recorded as `macros/NN-*.FCMacro`.
* Derived values are computed inside the VarSet by expression. Sketches bind to the
  derived names, so the formulas live in exactly one place.

## VALIDATION

1. `python3 scripts/audit_parametric.py` is clean.
2. **Flex test** (a binding alone doesn't prove the Param drives geometry): set
   `InlayDiameter` 150 → 100, recompute, measure the hole edge radius from the shape, and
   confirm it equals `TemplateHoleModelD/2`. Then restore the value.
3. `part_check_shape` / `validate_object` report a valid, closed solid; bounding box is centered on the origin.
4. Assert `TemplateThick > BushingProtrusion`, and `TemplateSide` ≤ 350 mm (K2 Plus bed).
   Exceeding the bed is a plan revision (see SIZE SET), not a Param tweak.
5. Physical: print the **Ø100** first (smallest, cheapest) in the production material, not PLA,
   because the comp values are material-specific. Measure the hole on two axes with calipers, set
   slicer shrinkage from the measured scale error, then tune `TemplateHolePrintComp` for any remaining
   constant offset. Check the plate's flatness.
   Rout a test recess in scrap and dry-fit a laser-cut disc, then tune `InlayFitClearance`/`LaserKerf`.

## MATERIAL

Candidates (Bradley, 2026-10-08): **PA6-CF12** (brand TBD) or **Polymaker Fiberon PETG-rCF08**.

* **Shrinkage scales with size.** On a 347 mm plate, even 0.2–0.3% XY shrink moves the Ø212 mm hole
  by ~0.5 mm diametral, which is far more than `InlayFitClearance`. A constant per-side offset can't
  correct a percentage error. So:
  - **Percentage shrink is fixed in the slicer's per-filament XY shrinkage compensation**, not in CAD.
    The CAD model stays true-size, and one STL serves both materials. *(Assumes Creality Print/Orca
    exposes per-filament XY shrinkage. Verify in the slicer before relying on it.)*
  - `TemplateHolePrintComp` covers only the constant "printed holes come out small" effect. Its
    value is **per material**. Re-tune it when switching filaments, and record the value used in
    the CLAUDE.md print profile.
* **Recommendation: PETG-rCF08 for the templates.** PA6 absorbs moisture after printing and
  swells, so a nylon hole's size drifts with the shop's humidity, and the fit would only be right on the day
  it was measured. PETG-rCF barely absorbs water. PA6-CF is tougher and more heat-resistant, but
  neither matters for a hand-router template.
* Both are carbon-fiber filled and abrasive, so they need a **hardened nozzle** (confirm what the K2 Plus has fitted).
  Both need **drying** before printing (PETG-rCF08: 65 °C / 3 h per the Polymaker TDS).
* **Flatness matters**: a warped plate tilts the router and tapers the recess wall. Large flat plates
  are the warp-prone case, so print in the enclosed chamber and check flatness with a straightedge
  before use.

## SIZE SET

Planned variants: **Ø100, Ø150, Ø200, Ø250** (Bradley, 2026-10-08). Every size comes from the same
model by changing `InlayDiameter`. A `macros/NN-export_all_sizes.FCMacro` will loop through the set,
recompute, export `stl/RecessTemplate-D<size>.stl`, and restore the original value.

Bed fit on the K2 Plus 350 mm bed (Milwaukee base, placeholder bushing/bit; hole = disc + 12.65, margin 67.09):

| Disc | Hole | Plate | Fit |
|---|---|---|---|
| 100 | 112.65 | 246.82 | ✅ 103 mm spare |
| 150 | 162.65 | 296.82 | ✅ 53 mm spare |
| 200 | 212.65 | 346.82 | ⚠️ fits with only 3.2 mm spare, so no brim; confirm the K2 Plus's real printable area |
| 250 | 262.65 | 396.82 | ❌ 46.8 mm over |

**Ø250 needs a strategy (Open Question 7).** A round outline doesn't help, since its diameter equals the square's side.
The Ø200 margin is so thin that a larger `BushingOD` or `BaseSupportOverlap` would push it over the bed.

## OPEN QUESTIONS (need answers before approval)

1. ~~Router~~: **Milwaukee M18 FUEL compact router (2723-20)**, using its 5-3/4" template sub-base (Porter-Cable-style
   guides, 1-3/16" hole); answered 2026-10-08. Still open: which bushing OD, its protrusion length, and the bit diameter.
   Candidate kit: Alocs "71333" brass inlay kit (bushing + snap-on collar + 1/8" downcut spiral on a 1/4" shank +
   centering pin, sized for 1/4" templates). Its bushing/collar ODs aren't published, so **measure with calipers on arrival**:
   bushing OD, collar OD, protrusion below the base, and actual bit diameter. Plan to run **collar off**
   (the collar is only for routing a plug, and laser-cut discs don't need one; the smaller OD also shrinks the plate).
2. ~~Sizes~~: **Ø100/150/200/250** (answered 2026-10-08). Still open: disc thickness. Disc thickness sets the router
   plunge depth. It doesn't drive template geometry, so it gets no Param unless something uses it.
3. Laser: does the laser software compensate for kerf, and what's the measured kerf?
4. Workholding: double-sided tape only, or add countersunk screw/clamp holes?
5. Engrave the size label (e.g. "Ø150") on the top face?
6. Plate shape: a solid square (simplest, up to ~347 mm print), or a round/ring outline with the same
   margin (less plastic and print time; alignment notches stay on the X/Y axes)?
7. How to handle Ø250, which exceeds the bed (see SIZE SET):
   - (a) **Segmented template**, recommended: 2 or 4 keyed segments, with a `SegmentCount` Param. The
     bushing crosses the seams, so joint registration needs its own clearance Param.
   - (b) Shrink the margin on Ø250 only. It would need a 43.7 mm margin vs the 67 required, so the
     router base would overhang the plate by ~23 mm at the outer edge.
   - (c) A narrow printed ring around the hole, with a separate sub-base to steady the router.
