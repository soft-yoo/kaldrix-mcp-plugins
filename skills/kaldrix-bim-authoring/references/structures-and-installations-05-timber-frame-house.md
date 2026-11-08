## Playbook: Domek drewniany szkieletowy (timber-frame house)

2. Wall build-ups: with `set_building_construction technology: TIMBER_FRAME` the estimate already measures walls and
   floors without a build-up with the technology defaults (`list_build_ups` → `technologyDefaults`: external
   `TIMBER_FRAME_WALL_15`, internal load-bearing `TIMBER_FRAME_INTERNAL_WALL_12`, partitions `TIMBER_FRAME_PARTITION_10`,
   floors between storeys `TIMBER_JOIST_FLOOR_OSB`; estimate rows noted "Domyślne z technologii"). Confirm them with
   `set_technology_build_ups` (one call per building), or pick other templates: `list_build_ups` with
   `includeTemplates=true` → `create_build_up_from_template` → `buildUpId`s.
3. `assign_build_up` with `elementType: "WALL"` and the wall ids (thickness becomes the sum of layers) for any wall that
   should differ from the default.
4. Doors and windows with `add_opening_by_intent` (before or after — the frame follows).
5. `get_wall_frame` (per floor or `wallId`) → studs, plates, king studs, trimmers, headers, sills, cripples with counts,
   metres and m³; read `warnings`.
6. Floor / ceiling: slab build-up `TIMBER_JOIST_FLOOR_OSB` (or `TIMBER_CABIN_CEILING_BOARDS`) on the room slabs from
   `get_floor_detail` slabs via `assign_build_up` `elementType: "SLAB"` → `get_slab_frame`; **or** a `TIMBER_FLOOR_JOISTS` framing
   system per room or for the whole floor. Each joist is counted once and always by a framing system: within a system
   outline that system, elsewhere the timber floor assumed from the slab build-up (section and spacing from the template,
   `assumedFramingSystemIds` in `get_slab_frame`; INFO `SLAB_FRAME_JOISTS_IN_FRAMING_SYSTEM`). To change section, spacing or
   direction of all joists of a slab, `create_framing_system` `TIMBER_FLOOR_JOISTS` for the whole floor (no spaceId/outline).
7. Roof: inspect `list_structures` first: an attic may already have its default roof. Otherwise `create_roof` →
   `preview_framing_system` `kind: "ROOF_RAFTERS"`. Read the suggested section and spacing: the backend chooses them
   from the span and covering; do not assume one fixed rafter section. Counter battens and battens are on by default.
   Send the reviewed preview version to `create_framing_system`, then use `get_framing_drawing` to describe it.
8. Estimate: playbook "Kosztorys bez podwójnego liczenia" below (wall frame `WALL_FRAME_ROLE_*`, layers
   `WALL_LAYER_AREA`, roof `FRAMING_ROLE_*`).
