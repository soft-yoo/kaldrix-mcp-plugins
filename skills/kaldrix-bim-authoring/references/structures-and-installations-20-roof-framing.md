## Roof framing

Detailed framing previews and authoring, including ROOF_RAFTERS, require STRUCTURES. Basic roof geometry and takeoff
stay available with project access; saved framing reads retain existing model data ([plan and module access](plan-and-module-access.md)).

1. Read the project, keep its `modelVersion`; set the building outline and create storeys first. `create_roof` on the
   highest building storey with the reviewed version and an idempotency key: `FLAT`, `SHED`, `GABLE`, `HIP`,
   `HALF_HIP` or `MANSARD`; the effective storey outline is the footprint (rooms do not define the roof).
   - Rectangular outline only: `HALF_HIP` takes `halfHipDepth` (omit for default); `GABLE` takes `secondPitchDegrees`
     for an asymmetric roof (`pitchDegrees` alone in `update_roof` makes it symmetric again).
   - `MANSARD`: `mansardLowerPitchDegrees` (45-80, default 60), `mansardBreakHeight` (break above the wall line,
     default 2.2 m); `pitchDegrees` is the gentler upper pitch. Eaves overhang may drop at most 0.5 m (0.28 m at 60°;
     default follows the lower pitch). No rafters generated (warning); framing counts only counter battens, battens
     and fascia boards.
   - `FLAT` may take `parapetHeight` (0.3-1.5 m above the roof top; 0 removes it): outer walls rise above the roof,
     slab between them, eaves overhang must be 0. Other kinds refuse it (`BIM_ROOF_PARAPET_*`). A raised wall keeps
     its height (`BIM_ROOF_PARAPET_WALL_LOCKED`): change `parapetHeight` with `update_roof`.
   - Keep `roofAssemblyId` (`list_structures` lists roof assemblies too). Never remove a wall created/adjusted by a
     roof on its own, even with the ROOF part disabled: `BIM_ROOF_WALL_MANAGED_BY_ASSEMBLY` requires removing the whole roof first.
2. `preview_framing_system` `kind = ROOF_RAFTERS` (+ `roofAssemblyId` when the floor has several roofs). Read current
   defaults and inspect the preview before choosing sections, spacing and supports.
   - Parametric: wall plates, collar ties, optional ridge beam. Battening: a counter batten on every rafter
     (`counterBattens`) and battens parallel to the eaves every 0.33 m along the slope (`battens`, `battenSpacing`
     0.15–0.5, `battenWidth`, `battenHeight`). Battening members have level `COVERING`: separate BOM lines for battens
     and counter battens (`FRAMING_ROLE_LENGTH`, services filtered to roles `BATTEN` / `COUNTER_BATTEN`), not in roof
     timber volume or piece count.
   - Fascia boards on eaves and barge boards on gable edges 2.5×20 (`fasciaBoards`, default on): roles `FASCIA` /
     `BARGE_BOARD`, level `COVERING`, priced in metres ("Deska czołowa i wiatrowa").
   - Geometry: rafters evenly spaced (≤ `mainSpacing`), outermost beside the barge board; opposite rafters meet in the
     vertical plane under the ridge; jack rafters end at the side of the hip / valley rafter; wall plates sit on the
     wall crown (outer edge flush with the wall face) and extend under the outermost rafter of a gable overhang; a shed
     roof gets a wall plate on the high wall too; collars at `collarHeightRatio` of the rise from wall plate to ridge.
   - Wall plates are counted in stock lengths with joints. Carpentry connectors (rafter–plate, collar bolts, ridge
     pair, jack–hip) = `FRAMING_CONNECTOR_COUNT` ("Łączniki ciesielskie więźby"); wall plate anchors to the ring beam =
     `FRAMING_FIXING_COUNT` ("Kotwy murłat i belek stropu") — two different items.
   - A pitched roof without framing gets default roof framing in the takeoff (`assumed`); with a roof build-up its
     rafter, counter-batten and batten layers are counted by that framing (layer volume 0, `countedInFramingSystemIds`).
   - Read preview `warnings` and support checks. The generator supports purlins with posts and braces (configure
     `purlins`, section, post spacing, brace parameters). Layout and quantity report do not certify structural
     adequacy; resolve or explain warnings.
3. Save with `create_framing_system`, the reviewed model version and an idempotency key. One roof holds one roof
   framing; revise via `update_framing_system`, not another create. Keep it parametric (the user edits spacing,
   sections, supports and covering in the app configurator); never create generated rafters separately. Extra manual
   beams: `add_structural_members` + `array`; they stay independent and need position/support checks after roof changes.
4. `get_framing_drawing` (`systemId`, or kind + scope for a preview) explains the result like a technical drawing:
   section across the ridge with every element named in English (Rafter, Collar tie, Wall plate, Counter batten,
   Batten, Ridge beam), roof pitch, ridge / eave / collar heights, overhang and plan spacing chains; tell the user what
   goes where from its `description`. Also works for ceilings and joists (section across main members: hangers, main
   and secondary CD profiles, UD profile, plasterboard, ceiling height, profile spacing).

Use `explode_framing_system` only when the user explicitly requests conversion to individually editable members: it
turns the system into plain structural members and removes the system and its fixings, connectors, boards and
FRAMING_* quantities, so it needs `confirm=true` and `projects.delete`. `remove_framing_system` deletes the system.

`get_element` framing references may target `ROOF_ASSEMBLY` metadata, not a kind accepted by `list_elements`; inspect
roof assemblies with `list_structures`. Missing purlin or wall-plate faces remain flagged settings: review and correct
the support plan. A converted installation circuit no longer exists; its lines keep geometry and point links but no
historical `circuitId`. Deleted point links are cleared.
