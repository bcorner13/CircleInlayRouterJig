# Intent — CircleInlayRouterJig

> HUMAN INPUT. Captured 2026-10-08 from Bradley's answers in the bootstrap session.
> Edit freely — plan.md is derived from this file, not the other way round.

Goal:
Parametric router template for cutting a circular inlay recess with a guide bushing.
The inlay disc itself is **laser cut**, so the jig only has to rout a recess
whose diameter matches a laser-cut disc of known nominal diameter.

Constraints:

* Must follow CAD_STANDARDS.md
* Used with a router guide bushing (bushing rides the template edge)
* Recess size must be derivable from the inlay diameter + tooling + laser kerf,
  so one knob (inlay diameter) re-sizes the template
* Printable without supports
* Target inlay diameters: **100, 150, 200, 250 mm** (confirmed 2026-10-08); 150 mm is the first build
* Router: **Milwaukee M18 FUEL compact router (2723-20)**, 5-3/4" template sub-base, Porter-Cable-style guides
* Bushing OD, bushing protrusion, bit diameter: **TBD — Bradley to specify**
