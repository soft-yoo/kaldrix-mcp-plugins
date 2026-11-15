## Geometry changes and opt-in installation automation

`configure_floor_installation_automation` enables underfloor replanning and/or compatible POWER/LIGHTING circuit
merging for a floor; mark existing circuits explicitly with `automaticallyManaged=true`. Needs INSTALLATION_DESIGN
(depends on INSTALLATIONS). Manual routes and loop metre targets are excluded; manual edits disable automatic
regrouping for that circuit. Water, drainage and radiators are never regrouped.

The user controls `maxLoopArea` (30 m² by default; a planning preference, not a design standard),
`maxElectricalPoints` and each loop's pipe length limit; area and length both constrain loop division. Underfloor
loops are replanned only where they already existed. Compatible loops can merge after rooms merge; different
manifolds, pipe specs or settings stay separate. Dedicated electrical appliances keep individual supplies. If manifold
outlets, passages or pipe limits prevent a valid layout, the whole geometry mutation fails atomically. A priced loop
is never silently removed from an estimate.

`set_building_outline` or `set_floor_outline` with `width` and `depth` resizes the layout proportionally: X width 10 →
11 m multiplies X positions by 1.1, keeping Y and height. Roofs, partitions, device locations and routes follow the
footprint; physical sections and device dimensions stay fixed. Re-read the full model and quantities afterwards.

### Customize one generated roof member section

1. Create the roof and its `ROOF_RAFTERS` system via the normal roof recipe workflow.
2. `get_framing_members(projectId, floorId, systemId, limit, cursor)` inspects generated members: a page of stable
   member IDs, the reviewed `modelVersion` and `inactiveMemberIds`. Follow `nextCursor` until absent. Never select by
   list index or write with IDs from an unsaved preview.
3. `update_framing_member` with that `expectedModelVersion`, the saved member ID and any of: width/height in metres
   (0.01–1), rafter eaves/ridge end `startExtension`/`endExtension` in metres, `removed: true`, or `collarHeightRatio`
   (0.2–0.9) for a collar tie pair. Generated axis, role, material and connections stay unless removed.
4. Review the returned layout/BOM, `get_framing_bill_of_materials` and the estimate quantity source. A stale write
   needs a fresh read and review; never fetch a new version just to retry blindly.
5. `reset=true` restores the recipe section and removes an inactive stored override.

Changing roof geometry, spacing, recipe or openings may produce different axes. An unmatched stored override stays
inactive with a warning and never transfers to another member by proximity or list index: read again, explicitly reset
old IDs, then customize the intended new axes. `explode_framing_system` only when manual endpoints or other
independent member properties are needed; section customization needs no explode. A successful geometry update implies
no structural capacity approval.
