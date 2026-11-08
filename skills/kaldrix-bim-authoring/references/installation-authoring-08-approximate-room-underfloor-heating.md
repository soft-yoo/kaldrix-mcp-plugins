## Approximate room underfloor heating

Use `preview_underfloor_heating` then `create_underfloor_heating` with identical intent and the preview modelVersion.
The backend draws SPIRAL or MEANDER as straight polyline segments with schematic corners.
It partitions orthogonal rooms (including rotated L/U outlines), subtracts exclusions and floor openings,
assigns free manifold outlets, and estimates both approaches separately through actual doors/cutouts.
Missing passages, unavailable outlets, invalid fields and stale versions block writes. Approaches are indicative;
no bend arcs, physical bundle packing or port fanout are calculated. Do not present this as an installation design.

Defaults: PE_X_16, SPIRAL, spacing 0.2 m, edge offset 0.1 m, full-loop limit 90 m, reserve 0.05,
manifold outlet count 12. These are estimating settings, not thermal or hydraulic sizing.
Generic PE_X_16, PE_RT_16 and PEX_AL_PEX_16 pipes with custom parameters are supported without a manufacturer ID.
`get_underfloor_pipe_products` provides optional versioned product identification and source data.
Selecting `pipeProductId` checks ID/spec compatibility; minimumBendRadius is informational.
Stored manifold dimensions do not change approximate paths or takeoff.

To adjust one loop, call `preview_underfloor_heating_update`, then `update_underfloor_heating` with its circuit UUID,
exactly its room in spaceIds, `editLoopOnly=true`, and optional `targetHeatingLength` (5–200 m).
The target is pipe metres in that loop's heating field, excluding both approaches and reserve.
Backend adjusts spacing within 0.05–0.5 m; loop boundaries, outlet and other loops stay unchanged.
The result exposes requested targetHeatingLength, measured heatingLength and effectiveSpacing.
The target is approximate: 50 m may produce about 55 m. Quantities always use measured segments.
Clear the target with editLoopOnly=true to restore the requested spacing.

In whole-room mode (`editLoopOnly=false`, no targetHeatingLength), parameters can repartition fields;
null targetLoopCount retains existing IDs and may add loops. An explicit targetLoopCount can remove loops.
Inspect removedCircuitIds, then use `reduce_underfloor_heating` with projects.delete and confirm=true.
Active estimate references block removal. A saved change has no undo; going back means restoring a saved project version (`preview_project_version`, then `restore_project_version` after the user's approval). New loop IDs require explicit estimate inclusion.

Every result includes English warning guidance: UNDERFLOOR_LAYOUT_APPROXIMATE marks schematic routes,
UNDERFLOOR_TARGET_LENGTH_DIFFERENCE reports a field-target difference above about 10 percent (minimum 0.1 m),
and UNDERFLOOR_MAX_LENGTH_EXCEEDED reports a manual loop exceeding the full installed-length limit.
These warnings do not block saving a manual loop edit, including spacing changes without a target field length.
The warning reports installed length and the configured threshold. The automatic splitter still enforces that limit.
Installed length = field + supply + return; purchase length = installed length × (1 + reserveFactor).
Do not multiply by two or add reserve again. Corners do not create pipe elbow fittings.
Frozen estimates change only on explicit refresh; preserve prices, markups and VAT.
The current layout rules and quantity engine use the same geometry for preview, save, read and drawings.
An invalid saved loop returns a finite diagnostic with no fictitious quantity; valid loops remain readable.
