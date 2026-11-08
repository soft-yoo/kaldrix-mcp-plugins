## Playbook: Sufit podwieszany

1. `preview_framing_system` `kind: "CEILING_GK_SINGLE"` (or `CEILING_GK_DOUBLE`, `CEILING_TIMBER`) with the room
   `spaceId`. **Omit `params.elevation`**: the ceiling goes under the structure above (room ceiling slab, a slab
   higher up, the roof, else the floor height) minus the 5 cm hanger gap and the ceiling depth — e.g. 2.48 m for a
   two-level GK grid under a slab whose underside is 2.60 m. `summary.structureUnderside` / `summary.hangerLength`
   show where the hangers end and how long they are.
2. `create_framing_system`. Ceiling points (`LIGHT_POINT_CEILING`, `DETECTOR_CEILING`, `SUPPLY_DIFFUSER` / `EXTRACT_DIFFUSER`) at the old
   ceiling height follow the new ceiling.
3. `validate_floor_geometry`: WARNING `FRAMING_CEILING_ABOVE_STRUCTURE` (grid reaches into the slab / roof or leaves
   < 5 cm) → run its action `LOWER_CEILING_UNDER_STRUCTURE` with `execute_lower_ceiling_under_structure_action`, or
   `update_framing_system` with `clearElevation=true`. Points with their own height then get
   `MOVE_POINT_TO_CEILING`.
4. Wool / vapour barrier above: ceiling build-up `SUSPENDED_CEILING_GK_WOOL` on the room's CEILING slab — the system
   keeps profiles, hangers and boards, the build-up only the wool and membrane.
