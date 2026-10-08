# Intent — CircleInlayRouterJig

> HUMAN INPUT. Captured 2026-10-08 from Bradley's answers in the bootstrap session.
> Edit freely — plan.md is derived from this file, not the other way round.

Goal:
Parametric router template for cutting a circular inlay recess with a top-bearing pattern bit.
The inlay disc itself is **laser cut**, so the jig only has to rout a recess
whose diameter matches a laser-cut disc of known nominal diameter.

Constraints:

* Must follow CAD_STANDARDS.md
* Used with a top-bearing pattern bit (the bearing rides the template wall; no guide bushing).
  Changed from a guide bushing on 2026-10-08.
* Recess size must be derivable from the inlay diameter + bit geometry + laser kerf,
  so one knob (inlay diameter) re-sizes the template
* Printable without supports
* Material: PA6-CF12 or Polymaker Fiberon PETG-rCF08 (production material for test prints too)
* Target inlay diameters: **100, 150, 200 mm** (confirmed 2026-10-08); 150 mm is the first build.
  250 mm is on the backlog (too big for the bed in one piece)
* Disc thickness 3.1–3.2 mm; laser kerf negligible (< .0007)
* Router: **Milwaukee M18 FUEL compact router (2723-20)**, 5-3/4" sub-base
* Bit: top-bearing downcut flush-trim, 3/8" cutter × 3/8" cutting length, 1/4" shank
  (Jiiolioa, Amazon). Exact dims to be measured on arrival
