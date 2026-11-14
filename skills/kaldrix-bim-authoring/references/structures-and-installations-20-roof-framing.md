## Roof framing

Detailed framing previews and authoring, including ROOF_RAFTERS, require STRUCTURES.
Basic roof geometry and takeoff remain available with project access. Saved framing reads
retain existing model data; see [plan and module access](plan-and-module-access.md).

1. Read the project and retain its `modelVersion`. Set the building outline and create its storeys first.
   Call `create_roof` on the highest building storey with the reviewed version and an idempotency key.
   Choose `FLAT`, `SHED`, `GABLE`, `HIP`, `HALF_HIP` or `MANSARD`; the effective storey outline supplies the footprint.
   Over a rectangular outline only: `HALF_HIP` takes `halfHipDepth` (omit for the default), and `GABLE` takes
   `secondPitchDegrees` for an asymmetric roof (`pitchDegrees` alone in `update_roof` makes it symmetric again).
   `MANSARD` takes `mansardLowerPitchDegrees` (45-80, default 60) and `mansardBreakHeight` (break above the wall line,
   default 2.2 m); `pitchDegrees` is the gentler upper pitch. The eaves overhang may drop at most 0.5 m
   (0.28 m at 60°; the default follows the lower pitch). Rafters are not generated for it (warning); the framing
   system counts only counter battens, battens and fascia boards.
   A `FLAT` roof may take `parapetHeight` (0.3-1.5 m above the roof top; 0 removes it): the outer walls rise above the
   roof, the slab sits between them and the eaves overhang must be 0. Other kinds refuse it (`BIM_ROOF_PARAPET_*`).
   A raised wall keeps its height (`BIM_ROOF_PARAPET_WALL_LOCKED`): change `parapetHeight` with `update_roof`.
   Rooms do not define the roof. Retain `roofAssemblyId`; `list_structures` also lists roof assemblies.
2. `preview_framing_system` with `kind = ROOF_RAFTERS` (and `roofAssemblyId` when the floor has several roofs).
   Read the current defaults and inspect the preview before choosing sections, spacing and supports.
   Wall plates, collar ties and an optional ridge beam are parametric. Roof battening includes
   a counter batten on every rafter (`counterBattens`) and battens parallel to the eaves
   every 0.33 m along the slope (`battens`, `battenSpacing` 0.15–0.5, `battenWidth`, `battenHeight`). Battening
   members have level `COVERING`: separate BOM lines for battens and counter battens (`FRAMING_ROLE_LENGTH`, priced by
   services filtered to roles `BATTEN` / `COUNTER_BATTEN`), not part of the roof timber volume or piece count.
   Fascia boards on the eaves and barge boards on the gable edges 2.5×20 (`fasciaBoards`, default on): roles
   `FASCIA` / `BARGE_BOARD`, level `COVERING`, priced in metres ("Deska czołowa i wiatrowa").
   Geometry rules: rafters are spaced evenly (≤ `mainSpacing`), the outermost one beside the barge board; opposite
   rafters meet in the vertical plane under the ridge; jack rafters end at the side of the hip / valley rafter;
   wall plates sit on the wall crown (outer edge flush with the wall face) and extend under the outermost rafter of
   a gable overhang; a shed roof gets a wall plate on the high wall too; collars sit at `collarHeightRatio` of the
   rise from the wall plate to the ridge. Wall plates are counted in stock lengths with joints; carpentry connectors
   (rafter–plate, collar bolts, ridge pair, jack–hip) are `FRAMING_CONNECTOR_COUNT` ("Łączniki ciesielskie więźby");
   wall plate anchors to the ring beam are `FRAMING_FIXING_COUNT` ("Kotwy murłat i belek stropu") — two different items.
   A pitched roof without framing gets the default roof framing in the takeoff (`assumed`); with a roof build-up its
   rafter, counter-batten and batten layers are counted by that framing (layer volume 0, `countedInFramingSystemIds`).
   Read the preview `warnings` and support checks. The generator supports purlins with posts and braces;
   configure `purlins` and the related section, post spacing and brace parameters. A generated layout
   and quantity report do not certify structural adequacy. Resolve or explain the reported warnings.
3. Save the previewed intent with `create_framing_system`, the reviewed model version and an idempotency key.
   One roof holds one roof framing. To revise an existing system use `update_framing_system`, not another create.
   Keep the system parametric: the user can edit spacing, sections, supports and covering in the app configurator.
   Do not create every generated rafter separately. Additional manual beams can be drawn or repeated with
   `add_structural_members` + `array`; they remain independent and need position/support checks after roof changes.
4. `get_framing_drawing` (`systemId`, or kind + scope for a preview) explains the result like a technical drawing:
   section across the ridge with every element named in English (Rafter, Collar tie, Wall plate, Counter batten, Batten,
   Ridge beam),
   roof pitch, ridge / eave / collar heights, overhang and plan spacing chains. Use its `description` to tell the
   user what goes where. It works for ceilings and joists too (section across the main members: hangers, main and
   secondary CD profiles, UD profile, plasterboard, ceiling height and profile spacing).

Only use `explode_framing_system` when the user explicitly requests conversion to individually editable members.
It turns a system into plain structural members for manual editing; it removes the
system and its fixings, connectors, boards and FRAMING_* quantities, so it needs `confirm=true` and
`projects.delete`. `remove_framing_system` deletes the system.
