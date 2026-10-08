# Project rules — CircleInlayRouterJig

This jig's whole value is one derivation chain: **laser-cut disc diameter → recess diameter → template hole diameter**. The hole is computed from `InlayDiameter`, `LaserKerf`, `InlayFitClearance`, `BushingOD`, `BitDiameter` and `TemplateHolePrintComp`. If any of those is shortcut to a literal, or the formula is copied into a sketch instead of living once in the VarSet, a re-size silently produces a recess the disc doesn't fit. Fix fit problems by tuning the right clearance knob, never by nudging the hole.

> **How to use this template:** Every `[FILL: …]` marker is a required edit. Leave none. A thin CLAUDE.md (one that just restates the global rules) is the failure mode this template exists to prevent — the point is to capture *this project's* specific topology, history, and current state, so the user does not have to re-explain it each session.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context. Cite the actual incident or constraint that motivates each one — generic restatements are useless because the global file already has them.

1. **Everything parametric.** The highest-risk sketch is `Sk_Hole`. Its diameter must bind to the *derived* `<<Params>>#VarSet.TemplateHoleModelD`, never to `InlayDiameter` directly and never to a hand-computed number. Derived values (`RecessDiameter`, `TemplateHoleDiameter`, `TemplateHoleModelD`, `TemplateMargin`, `TemplateSide`, `TemplateThick`) are expressions *inside the VarSet*, so the formula exists in exactly one place. No prior incident in this project (fresh as of 2026-10-08).

2. **No fixing geometry by editing raw sketch coordinates.** No prior incident in this project; rule applies preventively.

3. **Attach sketches to datum planes, not feature faces.** Base sketches go on `DatumPlane_Base`, top-face sketches (labels etc.) on `DatumPlane_Top` (offset `TemplateThick`). No prior DAG incident; rule applies preventively.

4. **Clearance concepts stay decoupled.** Three distinct physical concerns, one knob each:
   - `InlayFitClearance`: per-side glue gap between the laser-cut disc and the routed recess wall
   - `LaserKerf`: diameter the disc loses to the laser beam (laser process, not fit). Set it to 0 if the laser software already compensates
   - `TemplateHolePrintComp`: per-side FDM compensation for the printed hole coming out undersize (printer/material, not fit)
   - `BushingFloorGap`: vertical gap between the bushing tip and the workpiece (not a fit clearance; it drives `TemplateThick`)
   A recess that's too tight after a test rout is **not** fixed with `TemplateHolePrintComp`. Measure which stage is off first (printed hole vs. calipers, disc vs. nominal).

---

## Assembly architecture

There's no multi-part assembly. One printed part sits in a three-layer physical stack:

- **Workpiece** (bottom): the board receiving the inlay. The template is taped or clamped on top, with its 4 edge-midpoint `AlignNotch` notches aligned to layout lines through the intended recess center.
- **RecessTemplate** (middle): a square plate with a circular through-hole centered on the origin. The plate must be thicker than the bushing protrudes (`TemplateThick = BushingProtrusion + BushingFloorGap`), or the bushing drags on the workpiece.
- **Router** (top): a Milwaukee M18 FUEL compact router (2723-20) on its 5-3/4" template sub-base, which takes Porter-Cable-style guides. The base rides on the template's top face, and the **guide bushing rides the inside wall of the hole**. The bit, smaller than the bushing, cuts *outside* the bushing path, so the recess is larger than the bushing path and smaller than the hole: `Recess = Hole − BushingOD + BitDiameter`. The top-edge `LeadInChamfer` lets the bushing drop in.
- **Inlay disc**: laser cut separately (not printed, not modeled). It mates into the recess at `InlayFitClearance` per side.

There's no plug/male template. The laser makes the disc, which is why this isn't a classic two-template inlay kit.

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet: all parametric variables | — | ❌ not created yet; run `macros/00-bootstrap_params.FCMacro` after plan approval |
| `CircleInlayRouterJig.FCStd` | Body `RecessTemplate` | `Params.FCStd` | ❌ not created yet; awaiting plan.md approval |

No broken files. Neither FCStd exists yet. The project is scaffolded and its plan is pending approval.

---

## Params variables (summary)

See the PARAMETERS table in `plan.md` (authoritative until `Params.FCStd` exists, after which the VarSet itself is authoritative). Groups:
- **Inlay**: `InlayDiameter` (the size knob), `LaserKerf`
- **Tooling**: `BushingOD`, `BushingProtrusion`, `BitDiameter`, `RouterBaseDiameter`
- **Clearance**: `InlayFitClearance`, `TemplateHolePrintComp`
- **Template**: `BushingFloorGap`, `BaseSupportOverlap`, `LeadInChamfer`, `AlignNotchWidth`, `AlignNotchDepth`
- **Derived** (VarSet expressions, never hand-set): `RecessDiameter`, `TemplateHoleDiameter`, `TemplateHoleModelD`, `TemplateMargin`, `TemplateSide`, `TemplateThick`

Size set Ø100/150/200/250 (default 150). `RouterBaseDiameter` = 146.05 comes from the Milwaukee 2723-20 template base spec (sourced online; measure to confirm). `TemplateMargin` is **derived** (`RouterBaseDiameter/2 − BushingOD/2 + BaseSupportOverlap`): never hand-set it. Bushing and bit defaults are **placeholders** (5/8" bushing, 1/8" bit, 6 mm protrusion) until Bradley supplies them. **Ø200 fits the 350 bed with only ~3 mm to spare**, so any change that grows the margin or hole (bigger bushing, more overlap) breaks it. Re-check `TemplateSide` ≤ 350 for every size after tooling changes.

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
- Git: GitFlow (`main` + `develop`, `feature/*` off develop).

---

## Print profile

**No successful test print yet. Profile TBD.**
