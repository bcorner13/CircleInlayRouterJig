# CircleInlayRouterJig

Parametric, 3D-printed router template for cutting a circular inlay recess with a
guide bushing. It's sized to fit a **laser-cut** inlay disc. Built in FreeCAD 1.1.

Set the disc diameter (`InlayDiameter`) plus your bushing and bit, and the template
hole is derived automatically:

```
Recess       = InlayDiameter − LaserKerf + 2·InlayFitClearance
TemplateHole = Recess + BushingOD − BitDiameter
```

## Status

Scaffolded 2026-10-08. The design plan (`plan.md`) is awaiting approval, and no geometry exists yet.

## Layout

| Path | Contents |
|---|---|
| `Params.FCStd` | VarSet holding every parametric variable (created by `macros/00-bootstrap_params.FCMacro`) |
| `CircleInlayRouterJig.FCStd` | The template model |
| `intent.md` / `plan.md` | Design goal / approved modeling plan |
| `macros/` | FreeCAD macros that make all model changes |
| `scripts/audit_parametric.py` | Parametric-integrity audit; run it before every commit |
| `stl/`, `3mf/` | Print deliverables |
| `gcode/` | Slicer output (gitignored) |
| `images/` | Photos and renders |

## Usage

1. Open `Params.FCStd` in FreeCAD and set `InlayDiameter`, `LaserKerf`, `BushingOD`, `BushingProtrusion` and `BitDiameter`.
2. Recompute `CircleInlayRouterJig.FCStd` and export `RecessTemplate`.
3. Print, check the hole with calipers, and tune `TemplateHolePrintComp` if needed.
4. Tape the template to the workpiece, aligning the edge notches to your layout lines, then rout with the bushing riding the hole wall.
