## Geometry changes and opt-in installation automation

Use `configure_floor_installation_automation` to enable underfloor replanning and/or compatible POWER/LIGHTING circuit merging for a floor. Explicitly mark existing circuits with `automaticallyManaged=true`. INSTALLATION_DESIGN must be enabled (it depends on INSTALLATIONS). Manual routes and loop metre targets are excluded; manual edits disable automatic regrouping for that circuit. Water, drainage and radiators are never regrouped.

The user controls `maxLoopArea` (30 m² by default, a planning preference rather than a design standard), `maxElectricalPoints` and each loop's pipe length limit. Area and length both constrain loop division. Underfloor loops are replanned only where they already existed. Compatible loops can merge after rooms merge; different manifolds, pipe specs or settings remain separate. Dedicated electrical appliances retain individual supplies. If manifold outlets, passages or pipe limits prevent a valid layout, the entire geometry mutation fails atomically. A priced loop cannot be silently removed from an estimate.

Use `set_building_outline` or `set_floor_outline` with `width` and `depth` for proportional layout resizing: changing X width from 10 to 11 m multiplies X positions by 1.1, preserving Y and height. Roofs, partitions, device locations and routes follow the footprint; physical sections and device dimensions stay fixed. Re-read the full model and quantities after changes.

### Customize one generated roof member section

1. Create the roof and its `ROOF_RAFTERS` system using the normal roof recipe workflow.
2. Read `get_framing_members(projectId, floorId, systemId, limit, cursor)`. This returns a page
   of stable member IDs, the reviewed `modelVersion`, and `inactiveMemberIds`. Follow `nextCursor`
   until it is absent. Never select by list index or write using IDs from an unsaved preview.
3. Call `update_framing_member` with that `expectedModelVersion`, the saved member ID, and any of: width/height in
   metres (0.01–1), rafter eaves/ridge end `startExtension`/`endExtension` in metres, `removed: true`, or
   `collarHeightRatio` (0.2–0.9) for a collar tie pair. The generated axis, role, material and connections are
   retained unless the member is removed.
4. Review the returned layout/BOM, `get_framing_bill_of_materials`, and the estimate quantity source.
   A stale write requires a fresh read and review; do not fetch a new version just to retry blindly.
5. `reset=true` restores the recipe section. It also removes an inactive stored override.

Changing roof geometry, spacing, recipe or openings may produce different axes. An unmatched
stored override stays inactive with a warning and never transfers to another member by proximity
or list index. Read again, explicitly reset old IDs, then customize the intended new axes.
Use `explode_framing_system` only when manual endpoints or other independent member properties
are needed; section customization does not require exploding the system. No structural capacity
approval is inferred from a successful geometry update.

Use `get_framing_members` to inspect generated framing members and their stable keys before editing an individual member with `update_framing_member`.
