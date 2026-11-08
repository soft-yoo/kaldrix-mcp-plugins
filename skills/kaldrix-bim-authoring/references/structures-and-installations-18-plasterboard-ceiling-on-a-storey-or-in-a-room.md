## Plasterboard ceiling on a storey or in a room

1. `preview_framing_system` with `kind = CEILING_GK_SINGLE` (or `CEILING_GK_DOUBLE`, `CEILING_TIMBER`) and the
   room `spaceId`, or omit both `spaceId` and `outline` for the current whole-storey footprint.
   Use an explicit `outline` only for a custom field. Whole-storey scope follows later floor resizing. Show the user the summary (area, CD/UD lengths, hangers, `structureUnderside`, `hangerLength`) and
   the BOM lines. Change `params` (for example `{"mainSpacing":0.4,"boardLayers":2,"boardType":"GKBI"}`) until it
   fits; leave `elevation` out unless the user gives a height (default = under the structure above).
2. `create_framing_system` with the same input → keep the returned `systemId`.
3. A room holds one ceiling and one joist system. An implicit whole-storey scope also holds at most one ceiling
   and one joist system; `CEILING_GK_SINGLE`, `CEILING_GK_DOUBLE` and `CEILING_TIMBER` share the ceiling category.
   A duplicate fails with `FRAMING_DUPLICATE` — use `update_framing_system` instead. A room system may coexist
   with a whole-storey system, and explicit custom outlines remain independent scopes. This does not imply
   automatic subtraction of overlapping systems from quantities; inspect each system's scope and takeoff.
   On MCP updates and previews with `sourceSystemId`, only supplied non-null `params` fields replace stored values.
   Omit the `params` object to keep the stored set. To reset omitted values to kind defaults, send an intended
   replacement object with `replaceParams=true`. On `update_framing_system`, `clearElevation=true` resets only
   the stored height.
   For an existing GK ceiling, `kindChangeTargets` lists supported changes between `CEILING_GK_SINGLE` and
   `CEILING_GK_DOUBLE`; it does not guarantee permission or valid parameters. Preview with `sourceSystemId`, the
   target `kind` and the unchanged scope, then send that kind to `update_framing_system` with the reviewed model
   version. UUID, scope and explicit parameters remain unchanged; omitted stored values use the target defaults.
   A stored zero mounting profile spacing must be corrected before changing to DOUBLE
   (`BIM_FRAMING_KIND_CHANGE_SECONDARY_SPACING_REQUIRED`). Active estimate scopes block kind changes
   (`BIM_FRAMING_KIND_CHANGE_ESTIMATE_REFERENCED`): ask the user to remove that ceiling from the estimate scope
   first. Never delete and recreate a ceiling to bypass this protection. Other kind and scope changes remain unsupported.
4. `TIMBER_FLOOR_JOISTS` works the same way for a timber floor over a room. Joists span the shorter side; clear spans
   over 3 m get `BLOCKING` rows (every ≤ 2.5 m). Leave `mainWidth` / `mainHeight` out: the preview `suggestion` picks the
   section from the clear span (≤ 4 m 6×18, ≤ 5 m 6×20, ≤ 6 m 8×24, every 0.6 m); a weaker explicit section warns
   (`FRAMING_BELOW_SUGGESTED`) and spans over 6 m need a beam or an engineer.
5. Suggestions (`suggestion.params` + English `suggestion.reason`) are rules of thumb — say „orientacyjnie, sprawdź
   z projektem”. Roof framing picks the rafter section from the span and `coveringType` (clay tile default; metal sheet
   = wider spacing); a GK ceiling in a bathroom / WC / laundry gets CD every 0.40 m and a GKBI board. Fix a weaker
   explicit choice with the `APPLY_FRAMING_SUGGESTION` geometry action.

Ceiling slab openings are deducted by the backend within the system's actual scope, including openings that
cross a room boundary or reach its edge. Profiles and opening-edge framing are clipped to the remaining field;
the board surface may become notched or split into several regions. Read the net summary `area`, `boardArea`
(net area × board layers) and BOM. The preview `outline` is the scope boundary, not a replacement for net area;
the compact MCP summary intentionally omits the app renderer's board regions and void geometry. Edit the host
slab openings with the slab-opening tools, not by inventing framing members or subtracting quantities yourself.

Ceiling installation points (`LIGHT_POINT_CEILING`, `SUPPLY_DIFFUSER` / `EXTRACT_DIFFUSER`, `DETECTOR_CEILING`) use
the effective ceiling at their position: a room ceiling takes precedence over a custom-outline ceiling, then
the whole-storey ceiling. Without an applicable ceiling system, points use the floor height. Creating, changing
or removing a ceiling recomputes that height. Only ceiling-mounted points that sat at their previous effective height (±1 cm) move;
points with a custom height stay where they are.
