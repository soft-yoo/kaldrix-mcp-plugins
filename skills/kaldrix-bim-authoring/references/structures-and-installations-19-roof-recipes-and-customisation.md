## Roof recipes and customisation

`kaldrix://projects/roof-construction-recipes` holds editable starting points shared with the app. The resource is
unscoped: a listed recipe is not a promise that it fits the current roof.
- Choose roofType, pitchDegrees and eavesOverhang; create roof surfaces from the highest storey outline; then preview
  ROOF_RAFTERS with the recipe framingParams, adjust spacing, sections and supports, inspect warnings and save at the
  reviewed model version. A flat recipe has null framingParams: no automatic rafter system implied.
- Later customisation: update the parametric system; add extra manual members separately. Convert the system to
  individual members only after explicit user confirmation (changes takeoff, removes the automatic system). Never
  claim manual members regenerate with later roof edits.

`purlinRows` is `1` by default (also when stored as null): automatic support plans place that row halfway from the
eave wall face to the ridge. `purlinRows: 2` places two rows per slope at one-third and two-thirds of that plan run;
it currently requires an axis-aligned rectangular `GABLE` footprint, exactly two pitched quadrilateral faces and an
actual eave-wall-to-ridge plan run of at least 2 m on each face. These checks also apply while `purlins: OFF`; two rows
do not enable purlins by themselves. Preview first: invalid counts, unsupported geometry and short runs return
`BIM_FRAMING_PURLIN_ROWS_INVALID`, `BIM_FRAMING_PURLIN_ROWS_GEOMETRY_UNSUPPORTED` and
`BIM_FRAMING_PURLIN_ROWS_SPAN_TOO_SHORT`. An existing two-row system also blocks an incompatible roof or outline change
before any model write. Layout option only, not a load calculation or structural certification.

`purlinRows` controls automatic row generation. A saved `CUSTOM` support plan keeps its explicit purlins, post
positions and row offsets when parameters or recipes change. Read `get_roof_support_plan` first and explicitly choose
to preserve that plan or replace it with an automatic one; never silently discard supports to make a recipe match.
Returning to automatic generation needs `update_roof_support_plan` with `clear: true` after user confirmation (below).
