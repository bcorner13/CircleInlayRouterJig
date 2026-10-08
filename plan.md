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
| `BushingOD` | Length | 15.875 mm **TBD** | Tooling | Guide bushing outer diameter (5/8") |
| `BushingProtrusion` | Length | 6.0 mm **TBD** | Tooling | How far the bushing sticks out below the router base |
| `BitDiameter` | Length | 3.175 mm **TBD** | Tooling | Router bit cutting diameter (1/8") |
| `TemplateHolePrintComp` | Length | 0.15 mm | Clearance | Per-side FDM compensation, since printed holes come out undersize. Tune after test print |
| `BushingFloorGap` | Length | 1.5 mm | Template | Clearance between bushing tip and workpiece |
| `TemplateMargin` | Length | 75 mm **TBD** | Template | Plate material outside the hole on each side (router-base support: should be ≈ router base radius; 75 assumes a ~150 mm base) |
| `LeadInChamfer` | Length | 0.8 mm | Template | Top-edge chamfer on the hole so the bushing drops in cleanly |
| `AlignNotchWidth` | Length | 3 mm | Template | V/slot notch at each edge midpoint, used to align to layout lines |
| `AlignNotchDepth` | Length | 3 mm | Template | Notch depth into plate edge |
| `RecessDiameter` | Length | *derived* | Derived | Expression, see Concept |
| `TemplateHoleDiameter` | Length | *derived* | Derived | Expression, see Concept |
| `TemplateHoleModelD` | Length | *derived* | Derived | Hole as modeled (includes print comp) |
| `TemplateSide` | Length | *derived* | Derived | `TemplateHoleDiameter + 2*TemplateMargin` |
| `TemplateThick` | Length | *derived* | Derived | `BushingProtrusion + BushingFloorGap` (the bushing can't bottom out on the workpiece) |

Clearance concepts are decoupled: glue fit (`InlayFitClearance`), laser process
(`LaserKerf`), and FDM print fit (`TemplateHolePrintComp`) each get their own knob.

Worked example (Ø150 disc, tooling placeholders): Recess = 150 − 0.15 + 0.10 = **149.95**;
Hole = 149.95 + 15.875 − 3.175 = **162.65**; modeled hole **162.95**; plate
**312.65 × 312.65 × 7.5**, which fits the K2 Plus 350 mm bed with ~37 mm to spare. The bushing path
radius is ~73.4 mm, so a ~150 mm router base reaches the hole center at the far side. That's why
`TemplateMargin` is ~75 mm (≈ base radius): the outer half of the base always has plastic under it.
It's a large print (~310 mm square). The ring or skeleton variant listed in Open Questions would cut that.

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
   With `TemplateMargin` = 75, that caps the hole at ~200 mm (Ø150 disc → 312.65 mm plate, fits). Beyond that, the plate needs
   a ring shape or a split, bolted design (a plan revision, not a Param tweak).
   Router-base support: the base must stay mostly on plastic while the bushing circles the
   hole, so `TemplateMargin` should be roughly ≥ half the router base diameter.
5. Physical: print in PLA, measure the hole with calipers, and tune `TemplateHolePrintComp`.
   Rout a test recess in scrap and dry-fit a laser-cut disc, then tune `InlayFitClearance`/`LaserKerf`.

## SIZE SET

Planned variants: **Ø100, Ø150, Ø200, Ø250** (Bradley, 2026-10-08). Every size comes from the same
model by changing `InlayDiameter`. A `macros/NN-export_all_sizes.FCMacro` will loop through the set,
recompute, export `stl/RecessTemplate-D<size>.stl`, and restore the original value.

Bed fit with placeholder tooling (hole = disc + 12.65 mm), on the K2 Plus 350 mm bed:

| Disc | Hole | Plate @ margin 75 | Plate @ margin 45 | Max margin that fits |
|---|---|---|---|---|
| 100 | 112.65 | 262.65 ✅ | 202.65 ✅ | 118.7 |
| 150 | 162.65 | 312.65 ✅ | 252.65 ✅ | 93.7 |
| 200 | 212.65 | 362.65 ❌ | 302.65 ✅ | 68.7 |
| 250 | 262.65 | 412.65 ❌ | 352.65 ❌ | 43.7 |

**Ø200/Ø250 fit is unresolved.** It depends on router base size (Open Question 1) and a
strategy choice (Open Question 7). A round outline doesn't help, since its diameter equals the square's side.

## OPEN QUESTIONS (need answers before approval)

1. Real hardware: router, router base diameter, bushing OD, bushing protrusion length, bit diameter.
2. ~~Disc diameter~~: **150 mm** (answered 2026-10-08). Still open: disc thickness, and any other
   target sizes. Should I export one STL per size (e.g. `stl/RecessTemplate-D150.stl`)? Disc thickness sets the router
   plunge depth. It doesn't drive template geometry, so it gets no Param unless something uses it.
3. Laser: does the laser software compensate for kerf, and what's the measured kerf?
4. Workholding: double-sided tape only, or add countersunk screw/clamp holes?
5. Engrave the size label (e.g. "Ø150") on the top face?
6. Plate shape: a solid square (simplest, ~310 mm print), or a round/ring outline with the same
   margin (less plastic and print time; alignment notches stay on the X/Y axes)?
7. Sizes that exceed the bed (see SIZE SET):
   - (a) **Segmented template**, recommended: 2 or 4 keyed segments, with a `SegmentCount` Param. The
     bushing crosses the seams, so joint registration needs its own clearance Param.
   - (b) Shrink the margin on big sizes only. This gives the router base less support, and it can't
     rescue Ø250 with a full-size router.
   - (c) A narrow printed ring around the hole, with a separate sub-base to steady the router.
