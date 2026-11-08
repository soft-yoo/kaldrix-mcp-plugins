### Quantity mapping

Never infer the measured object from the billing unit alone. `M3` may mean wall volume, room volume, slab volume, or a
manual quantity. Read `kaldrix://estimation/quantity-rules` and use one exact combination:

- wall painting: `WALL` + `WALL_NET_AREA` + `M2`;
- room wall finish: `SPACE` + `SPACE_WALL_AREA` + `M2`;
- floor finish: `SPACE` + `SPACE_FLOOR_AREA` + `M2`;
- concrete slab: `SLAB` + `SLAB_VOLUME` + `M3`;
- room cubature allowance: `SPACE` + `SPACE_VOLUME` + `M3`;
- facade insulation: `WALL` + `WALL_EXTERIOR_NET_AREA` + `M2` with `elementSubtypes: ["EXTERNAL"]`;
- skirting boards: `SPACE` + `SPACE_SKIRTING_LENGTH` + `MB`;
- sloped attic ceiling (plasterboard on the roof underside): `SPACE` + `SPACE_SLOPED_CEILING_AREA` + `M2` — the flat part under
  the collar ties is the ceiling slab, priced separately;
- window sills / sealing tape: `OPENING` + `OPENING_WIDTH` / `OPENING_PERIMETER` + `MB` with `elementSubtypes: ["WINDOW"]`;
- gutters / ridge tiles / verge flashings: `SLAB` + `ROOF_EAVE_LENGTH` / `ROOF_RIDGE_LENGTH` / `ROOF_VERGE_LENGTH` + `MB`
  with `elementSubtypes: ["ROOF"]`; a roof edge against a higher wall (ground floor in front of the upper floor, garage at
  the house) is none of them — wall abutment flashing `SLAB` + `ROOF_WALL_FLASHING_LENGTH` + `MB` with `["ROOF"]`;
- layered build-ups (walls/slabs with `buildUpId`, see `list_build_ups`): plasterboard on timber-frame walls
  `WALL` + `WALL_LAYER_AREA` + `M2` with `elementSubtypes: ["GYPSUM_BOARD"]`; timber wall frame per member role
  `WALL` + `WALL_FRAME_ROLE_VOLUME` + `M3` with `["STUD","BOTTOM_PLATE","TOP_PLATE","KING_STUD","TRIMMER","SILL","CRIPPLE"]`
  and headers `["HEADER"]` (also `WALL_FRAME_ROLE_LENGTH` `MB` / `WALL_FRAME_ROLE_COUNT` `PCS`; roles required, see
  `get_wall_frame`), frame assembly `WALL` + `WALL_LAYER_AREA` + `M2` with `["TIMBER_FRAME"]`; timber floor joists (also
  from a slab build-up — the timber floor assumed from it, see `get_slab_frame` `assumedFramingSystemIds`)
  `FRAMING_SYSTEM` + `FRAMING_ROLE_VOLUME` + `M3` with `["TIMBER_FLOOR_JOISTS","JOIST","TRIMMER","HEADER","BLOCKING"]`,
  ceiling battens `FRAMING_SYSTEM` + `FRAMING_ROLE_LENGTH` + `MB` with `["CEILING_TIMBER","BATTEN"]` (slabs have no member
  rules — the framing systems count joists and battens); slab OSB / wool / foil / ceiling GK / boards `SLAB` + `SLAB_LAYER_AREA` + `M2` with
  `["OSB"]` / `["MINERAL_WOOL"]` / `["VAPOR_BARRIER"]` / `["GYPSUM_BOARD"]` / `["TIMBER_BOARDS"]`; Teriva `["BEAM_AND_BLOCK"]`;
  RC floor concrete `SLAB` + `SLAB_LAYER_VOLUME` + `M3` with `["REINFORCED_CONCRETE"]`;
  roof membrane `SLAB` + `SLAB_LAYER_AREA` + `M2` with `["ROOF", "ROOF_MEMBRANE"]`. These rules need at least one
  `LayerMaterial` in `elementSubtypes` (WallType/SlabType names may be added to narrow the elements); other rules
  reject LayerMaterial names. Area = single-face net wall area (openings subtracted) or net slab area per matching
  layer; elements without a build-up or without such a layer give 0;
- excavation not represented in BIM: omit `defaultElementType`, then use `MANUAL` + `M3`.

`elementSubtypes` narrows the scopes to subtypes of `defaultElementType`: `WallType` names for walls (`EXTERNAL` =
external walls), `SlabType` for slabs, `OpeningType` for openings, `InstallationType` for installations (e.g. `CABLE`),
`InstallationPointKind` for installation points (e.g. `SOCKET`), `StructuralMemberRole` for structural members
(e.g. `RING_BEAM`) and `FramingSystemKind` for framing systems (e.g. `CEILING_GK_SINGLE`). Points, members and
framing systems and their `FRAMING_*` bill of materials: `kaldrix://projects/structures-and-installations`.
Walls and slabs additionally accept `LayerMaterial` names (e.g. `GYPSUM_BOARD`, `MINERAL_WOOL`) — only with the
`WALL_LAYER_*` / `SLAB_LAYER_*` rules, where they pick the build-up layers to measure.
A service with a filter measures only its `defaultElementType` (no extra `M2` sources below) and rejects scopes of
another subtype.

The server rejects mismatched element type, quantity rule, and unit combinations. A BIM-linked recipe may still be
added with a manual `serviceQuantity` when the user does not want to use model geometry.

A service accepts more sources than its `defaultElementType`: `get_service_recipe` returns them as `quantitySources`.
Every `M2` service also accepts `WALL` (`WALL_NET_AREA`, one face), `SLAB` (`SLAB_AREA`) and `SPACE`, unless that type
is already its `defaultElementType` with its own rule. A room gives
the measure matching the service: its clear floor area (`SPACE_FLOOR_AREA`) for horizontal services whose rule is
`SLAB_AREA` or `SPACE_FLOOR_AREA` (screed, floor tiles, ceiling), and its wall finish area (`SPACE_WALL_AREA`) for
every other `M2` service. A saved measurement gives its own unit: a `LINE` measurement its length in `MB` (for `MB`
services), an `AREA` measurement its area in `M2` (for `M2` services, and for `M3` services with the layer thickness in
metres as `measurementMultiplier` in `plan_estimate_position_quantity`, e.g. 0.1 for 10 cm; always ask for the thickness — the server never guesses it and a
multiplier of 1 means a 1 m layer). Services billed in `PCS`, `SET`, `PACK`, `KG`, `TON`, `LITER`,
`HOUR` or `DAY` accept no measurement; `list_estimate_quantity_sources` does not offer one.

A mismatched scope is rejected with a code: `ESTIMATE_SCOPE_SOURCE_NOT_SUPPORTED` (the element type is not a source
of this service), `ESTIMATE_SCOPE_UNIT_MISMATCH` (`sourceUnit` differs from the service unit, or from `MB`/`M2` for a
measurement), `ESTIMATE_SCOPE_MEASUREMENT_NOT_SUPPORTED` (the measurement cannot give the service unit).
