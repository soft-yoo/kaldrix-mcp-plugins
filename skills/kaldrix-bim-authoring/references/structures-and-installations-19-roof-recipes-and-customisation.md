## Roof recipes and customisation

Read `kaldrix://projects/roof-construction-recipes` for editable starting points shared with the app. This MCP
resource is unscoped: a listed recipe is not a promise that it fits the current roof.
Choose roofType, pitchDegrees and eavesOverhang, then create roof surfaces from the highest storey outline.
After creation, preview ROOF_RAFTERS with the recipe framingParams. Adjust parameters based on the requested
spacing, sections and supports, inspect warnings, then save at the reviewed model version.
A flat recipe has null framingParams: it does not imply an automatic rafter system.
For later customisation, update the parametric system. Add extra manual members separately. Convert the
system to individual members only after explicit user confirmation; conversion changes takeoff and
removes the automatic system. Do not claim manual members regenerate with later roof edits.

`purlinRows` is `1` by default (also when stored as null); automatic support plans place that row halfway
from the eave wall face to the ridge. `purlinRows: 2` places two rows per slope at one-third and two-thirds
of that plan run. It currently requires an axis-aligned rectangular `GABLE` footprint, exactly two pitched
quadrilateral faces and an actual eave-wall-to-ridge plan run of at least 2 m on each face. These checks also
apply while `purlins: OFF`; two rows do not enable purlins by themselves. Preview before saving: invalid counts,
unsupported geometry and short runs return `BIM_FRAMING_PURLIN_ROWS_INVALID`,
`BIM_FRAMING_PURLIN_ROWS_GEOMETRY_UNSUPPORTED` and `BIM_FRAMING_PURLIN_ROWS_SPAN_TOO_SHORT`, respectively.
An existing two-row system also prevents an incompatible roof or outline change before any model write.
This is a layout option, not a load calculation or structural certification.

`purlinRows` controls automatic row generation. A saved `CUSTOM` support plan keeps its explicit purlins,
post positions and row offsets when parameters or recipes change. Read `get_roof_support_plan` first and
explicitly choose whether to preserve that plan or replace it with an automatic one; never silently discard
supports to make a recipe match. Returning to automatic generation requires `update_roof_support_plan`
with `clear: true` after user confirmation, as described below.
