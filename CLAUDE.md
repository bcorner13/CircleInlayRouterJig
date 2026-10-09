# Project rules — CircleInlayRouterJig

This jig's whole value is two derivation chains. **Diameter**: laser-cut disc → recess → template hole (`InlayDiameter`, `LaserKerf`, `InlayFitClearance`, `BearingOD`, `CutterDiameter`, `TemplateHolePrintComp`). **Height**: disc thickness + bit geometry → underside relief + plate thickness (`RecessDepth`, `CutterLength`, `BearingStackHeight`, `ReliefGap`, `CollarGap`). The height chain leaves the bearing only a ~3 mm band of wall to ride, so it's as fit-critical as the diameter chain. If any of those is shortcut to a literal, or the formula is copied into a sketch instead of living once in the VarSet, a re-size silently produces a recess the disc doesn't fit. Fix fit problems by tuning the right clearance knob, never by nudging the hole.

> **How to use this template:** Every `[FILL: …]` marker is a required edit. Leave none. A thin CLAUDE.md (one that just restates the global rules) is the failure mode this template exists to prevent — the point is to capture *this project's* specific topology, history, and current state, so the user does not have to re-explain it each session.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context. Cite the actual incident or constraint that motivates each one — generic restatements are useless because the global file already has them.

1. **Everything parametric.** The highest-risk sketch is `Sk_Hole`. Its diameter must bind to the *derived* `<<Params>>#VarSet.TemplateHoleModelD`, never to `InlayDiameter` directly and never to a hand-computed number. `Pocket_Relief.Length` binds to `ReliefHeight` and `Pad_Plate.Length` to `TemplateThick`, never to literals. Derived values (`RecessDiameter`, `TemplateHoleDiameter`, `TemplateHoleModelD`, `ReliefDiameter`, `ReliefHeight`, `TemplateThick`, `BearingBand`, `TemplateMargin`, `TemplateSide`) are expressions *inside the VarSet*, so the formula exists in exactly one place. No prior incident in this project (fresh as of 2026-10-08).

2. **No fixing geometry by editing raw sketch coordinates.** No prior incident in this project; rule applies preventively.

3. **Attach sketches to datum planes, not feature faces.** Base sketches go on `DatumPlane_Base`, top-face sketches (labels etc.) on `DatumPlane_Top` (offset `TemplateThick`). No prior DAG incident; rule applies preventively.

4. **Clearance concepts stay decoupled.** Distinct physical concerns, one knob each:
   - `InlayFitClearance`: per-side glue gap between the laser-cut disc and the routed recess wall
   - `LaserKerf`: diameter the disc loses to the laser beam (laser process, not fit). Set it to 0 if the laser software already compensates
   - `TemplateHolePrintComp`: per-side FDM compensation for the printed hole coming out undersize (per printer/material, not fit)
   - `ReliefClearance`: per-side radial gap between the spinning cutter and the underside relief wall
   - `ReliefGap`: vertical gap between the cutter top and the relief ceiling
   - `CollarGap`: vertical gap between the plate top and the bit's lock collar (the collar doesn't spin freely and would burn the plastic)
   A recess that's too tight after a test rout is **not** fixed with `TemplateHolePrintComp`. Measure which stage is off first (printed hole vs. calipers, disc vs. nominal).

---

## Assembly architecture

There's no multi-part assembly. One printed part sits in a three-layer physical stack:

- **Workpiece** (bottom): the board receiving the inlay. The template is taped or clamped on top, with the apexes of its 4 edge-midpoint **V notches** (90°, 8 deep × 16 wide) on a pencil crosshair drawn through the intended recess center. Each apex lies exactly on the hole's X/Y centerline.
- **RecessTemplate** (middle): a square plate with a circular through-hole centered on X/Y, with its bottom face (workpiece side) at z = 0. The hole has an **underside relief**, a larger counterbore `ReliefDiameter` × `ReliefHeight` cut up from the bottom, so the spinning cutter never touches plastic. Above the relief is the **bearing band** (`BearingBand`, ~3.2 mm), the only wall the bearing rides. Printed **as modeled, with the workpiece (bottom) face on the bed**. That keeps the fit-critical bearing band in the upper layers, away from elephant foot, and the label engraves cleanly in the top layers. The only overhang is the 0.85 mm relief ledge. Flat is also the strongest orientation. Plan rev 2 first said top face down; this was reversed at export time, see plan.md "Print orientation". `macros/03-export_print_files.FCMacro` exports Ø100/150/200 to `stl/` + `3mf/` in this orientation. It meshes at 0.005 mm deflection and asserts the meshed hole loses ≤ 0.01 mm, because the 0.1 mm default lost 0.036 mm on a Ø200 hole.
- **Router** (top): a Milwaukee M18 FUEL compact router (2723-20) on its 5-3/4" (146.05 mm) sub-base, with no guide bushing. The base rides on the plate's top face. The bit is a **top-bearing downcut pattern bit** (3/8" cutter × 3/8" length, two stacked bearings, then a set-screw lock collar). The bearing rides the bearing band, and the cutter below it cuts the recess wall on the bearing's path: `Recess = Hole − BearingOD + CutterDiameter`.
- **Inlay disc**: laser cut separately (not printed, not modeled). It mates into the recess at `InlayFitClearance` per side. Its thickness is `RecessDepth`.

There's no plug/male template. The laser makes the disc. **Usage constraint: rout in one full-depth pass.** The plate height assumes the cutter is at full `RecessDepth`. A shallower pass raises the cutter into the plate above the relief.

Design history: rev 1 of the plan used a guide-bushing inlay kit. It switched to the bearing bit on 2026-10-08 because the bearing bit has no bushing-concentricity error, simpler hole math, faster safe clearing with a 3/8" cutter, and a smaller plate. Rev 1 is in git history.

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet: 35 variables (9 derived by expression) | — | ✅ created by macro 00 (2026-10-08) |
| `CircleInlayRouterJig.FCStd` | Body `RecessTemplate` + top-level `Label_Text` ShapeString | `Params.FCStd` | ✅ built by macros 01 + 02; audit clean; flex-tested |

No broken files. Built 2026-10-08 and verified: the solid is valid; the volume matches the analytic value exactly (698,649.139 mm³ before the label); flex tests at Ø100/Ø200 and RecessDepth 4.2 measured equal to the Params. Not yet printed.

**Model tree** (`RecessTemplate`, tip `Pocket_Label`): `DatumPlane_Base` → `Sk_Plate` (R`PlateCornerRadius` corners drawn in the sketch)/`Pad_Plate` → `Sk_Hole`/`Pocket_Hole` (ThroughAll, Reversed) → `Sk_Relief`/`Pocket_Relief` (Reversed) → `Sk_Standoff`/`Pocket_Standoff` (optional ring foot; `Suppressed` is bound to `StandoffHeight == 0mm`) → `Sk_Notch` (V)/`Pocket_Notch` → `PolarPattern_Notches` (×4) → `DatumPlane_Top` → `Binder_Label` → `Pocket_Label`. `Label_Text` (Draft ShapeString, outside the body) is attached to `DatumPlane_Top`, and its `String` is the expression `<<Ø%g>> % (InlayDiameter / 1mm)`. All sketches sit at z = 0 on `DatumPlane_Base`, so every pocket from them needs `Reversed = True` to cut up into the plate (verified on 1.1.3: not reversed removes nothing).

**Rebuild order:** `01-build_template` with `REBUILD=True` deletes the body, which orphans the label objects. Always follow it with `02-add_size_label` `REBUILD=True`.

---

## Params variables (summary)

See the PARAMETERS table in `plan.md` (authoritative until `Params.FCStd` exists, after which the VarSet itself is authoritative). Groups:
- **Inlay**: `InlayDiameter` (the size knob), `LaserKerf`, `RecessDepth` (disc thickness; drives plate height)
- **Tooling**: `CutterDiameter`, `CutterLength`, `BearingOD`, `BearingStackHeight`, `RouterBaseDiameter`
- **Clearance**: `InlayFitClearance`, `TemplateHolePrintComp`, `ReliefClearance`
- **Template**: `ReliefGap`, `CollarGap`, `BaseSupportOverlap`, `AlignNotchWidth`, `AlignNotchDepth`, `PlateCornerRadius`
- **Standoff** (optional ring foot): `StandoffHeight` (0 = off), `FootRingWidth`; derived `FootRingDiameter`. With feet on, macro 03 exports **top face down** with a `-S<h>` file suffix, and asserts a plate web ≥ 4 mm (so ≤ ~6.5 mm with this bit). See plan.md OPTION section.
- **Label**: `LabelSize`, `LabelDepth`, `LabelInset` (bottom-left corner; outside the router-base sweep at every size, with the tightest margin at Ø100: 125.2 vs 118.3 mm)
- **Derived** (VarSet expressions, never hand-set): `RecessDiameter`, `TemplateHoleDiameter`, `TemplateHoleModelD`, `ReliefDiameter`, `ReliefHeight`, `TemplateThick`, `BearingBand` (validation only), `TemplateMargin`, `TemplateSide`

Size set Ø100/150/200 (default 150). **Ø250 is on the backlog** (plan.md BACKLOG): its plate would be ~390.6 mm, over the bed. `RecessDepth` = 3.2: discs are 3.1–3.2 mm thick, and the recess is sized for the thickest because the engraved discs can't be sanded flush. `LaserKerf` = 0: Bradley measured it at < .0007 in (0.018 mm), which is negligible. `RouterBaseDiameter` = 146.05 comes from the Milwaukee 2723-20 sub-base spec (sourced online; measure to confirm). The bit dims are listing values, and `BearingStackHeight` = 5.2 is **estimated from the listing drawing**. Measure them on arrival. Bed fit: Ø100/150/200 → 240.6/290.6/340.6 mm (Ø200 has ~9.4 mm spare). Asserts (the macro checks these): `BearingBand ≥ 2.5`, `TemplateSide ≤ 350`.

---

## How to verify your change didn't break parametric

After any FreeCAD edit, before considering the task done:

```bash
python3 scripts/audit_parametric.py
```

This script flags:
- Sketches with 0 constraints
- Sketches with dimensional constraints lacking expression bindings
- Sketches attached to feature faces (DAG risk)
- Params variables used nowhere (dead Params)

The script is authoritative. If it reports violations, fix them via a `macros/*.FCMacro` change before saving or committing — never by direct coordinate edits or FCStd XML surgery.

No exemptions; audit must be clean. Note: `scripts/audit_parametric.py` was copied from **Lockbox Bracket** rather than the canonical Spade copy, because that's the copy with all three known enum/property-list fixes (see the global memory `reference_audit_parametric_type_bug`). The audit still doesn't check `Placement`, `AttachmentOffset` or datum attachment. For the hole, also run the plan.md **flex test**: change `InlayDiameter`, then measure that the hole radius follows.

---

## Memory files (deeper context)

`~/.claude/projects/[encoded-path]/memory/MEMORY.md` indexes the persistent memories for this project. If you're unsure *why* a rule exists, read those files first — they trace each rule to a concrete past incident.

No project-scoped memories yet. This is a fresh project (bootstrapped 2026-10-08).

---

## Workflow notes

**Invariant (apply to every FreeCAD project — do not edit):**

- **Inspect/edit FreeCAD models via the MCP bridge — never with shell tools.** Do **not** `unzip`/`grep`/`cat`/`sed`/`strings`/etc. a `.FCStd`. Use the FreeCAD Robust MCP server: `get_connection_status` first, then `open_document`, `list_objects`, `inspect_object`, `execute_python`, and macros. This is **enforced** by a PreToolUse hook in `.claude/settings.json` (copied from the root `settings.template.json` during bootstrap) — raw shell access to `.FCStd` is blocked. Only fall back to read-only `unzip` if the MCP bridge is genuinely unreachable, and ask the user first.
- **MCP server auto-starts with FreeCAD.** If an `mcp__freecad__*` call fails, the right interpretation is "FreeCAD isn't running" — ask whether to launch it. Do **not** silently fall back to `unzip` + XML parsing, and do **not** retry the same MCP call.
- **Write changes as `macros/*.FCMacro` files**, not direct XML edits. Reasons: reviewable, re-runnable, idempotent-friendly, uses FreeCAD's own serialization.
- **Cross-document expressions**: use the canonical form `<<Params>>#VarSet.VarName`. The shorter `<<Params>>.VarName` form sometimes fails with "Params not found."
- **Run `python3 scripts/audit_parametric.py` before committing.** If it reports violations, fix via macro, not by editing FCStd XML.

**Project-specific:**

- Macros are numbered `NN-name.FCMacro` and symlinked into `~/Library/Application Support/FreeCAD/v1-1/Macro/` with the prefix **`CIRJ-`** (e.g. `CIRJ-00-bootstrap_params.FCMacro`).
- Multiple inlay sizes come from the same model by changing `InlayDiameter`. Export each one as `stl/RecessTemplate-D<diameter>.stl` rather than cloning bodies.
- The guard hook also blocks Bash commands that merely *mention* `.FCStd` alongside `cat`/`grep`/etc. (e.g. heredocs). Author such files with the Write tool.
- **Material**: PA6-CF12 or Fiberon PETG-rCF08 (PETG-rCF recommended, because PA6 swells with humidity). Percentage shrink is compensated **in the slicer filament profile**, never by scaling CAD values. `TemplateHolePrintComp` is per-material, so record it with the print profile. Abrasive CF filament needs a hardened nozzle.
- **Close other projects' `Params` documents first.** `<<Params>>#VarSet.…` resolves by document *label*, and other projects (e.g. MagicCardBox) also have a doc labelled `Params`. Macro 00 aborts if a foreign `Params` is open. Macros match this project's documents by **file path**, never by name.
- **Cross-document bindings need a saved owner.** FreeCAD 1.1.3 raises `Owner document not saved` when you `setExpression` to `<<Params>>` from an unsaved document. Macro 01 saves the empty model file before binding.
- **Label text format**: `str()` in expressions renders lengths as "150.0", so use `<<Ø%g>> % (… / 1mm)`. `int()` doesn't exist in FreeCAD expressions.
- Git: GitFlow (`main` + `develop`, `feature/*` off develop).

---

## Print profile

**No successful test print yet.**

Slicer settings live in `3mf/circleinlayjig-base.3mf`, a Creality Print 7.3 project Bradley saved from D150 on 2026-10-08. It has K2 Plus, filament slot 4 (generic CR-PETG), mouse ears (`brim_ears`), elephant-foot comp 0.15, and a variable-layer-height table (50 quality / 7 smoothness). Macro 03 copies that project for each size, swaps in the fine mesh, centers it on the bed, and applies `SLICER_OVERRIDES`: 4 walls, 5 top layers, `topmost` ironing, wipe tower off, brim ear max angle 150 (125 found no corners on the R10 plate). It also sets `EAR_WIDTH` 4.5 mm for D200, because 5 mm ears make it 350.8 mm on a 350 mm bed. The layer-height table is absolute z, so it's kept only while the plate is still 10.525 mm tall and not flipped. If the bit measurements change `TemplateThick`, re-save the base project (macro 03 warns and drops the table). To change print settings, edit the base in Creality, not the per-size files: those are regenerated.

**Creality Print 7.3 on macOS has no working CLI** (tested 2026-10-08). `--slice`, `--help` and `--logfile` are ignored, and it just opens the GUI. OrcaSlicer 2.3.2's CLI runs, but it rejects Creality project values (`wall_filament: 0` etc.). So verify each 3MF by opening it in the Creality GUI: ears at the corners, varying layer heights, no wipe tower.
