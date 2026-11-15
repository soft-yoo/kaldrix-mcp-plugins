## Approximate room underfloor heating

The full contract (layout rules, defaults, loop edits, warnings, quantities) is in `kaldrix://projects/installation-authoring`
([approximate room underfloor heating](installation-authoring-08-approximate-room-underfloor-heating.md)); read it before authoring loops. Key points:

- `preview_underfloor_heating` → `create_underfloor_heating` with identical intent and the preview modelVersion. SPIRAL
  or MEANDER loops are schematic straight polyline segments; estimating settings, not an installation design or
  thermal/hydraulic sizing. Missing passages, unavailable outlets, invalid fields and stale versions block writes.
- Pipes: generic PE_X_16 (default), PE_RT_16, PEX_AL_PEX_16; optional `get_underfloor_pipe_products` → `pipeProductId`.
- One loop: `preview_underfloor_heating_update` → `update_underfloor_heating` with `editLoopOnly=true` and optional
  `targetHeatingLength` (5–200 m); result has targetHeatingLength, heatingLength, effectiveSpacing.
- Whole room (`editLoopOnly=false`) may remove loops (targetLoopCount → removedCircuitIds): `reduce_underfloor_heating` with projects.delete
  and confirm=true. No undo: `preview_project_version`, then `restore_project_version` after the user's approval.
- Warnings UNDERFLOOR_LAYOUT_APPROXIMATE, UNDERFLOOR_TARGET_LENGTH_DIFFERENCE and UNDERFLOOR_MAX_LENGTH_EXCEEDED do not
  reject approximate estimating. The automatic splitter still enforces the configured limit.
- Installed length = field + supply + return; purchase length = installed length × (1 + reserveFactor). Never multiply
  by two or add reserve again. Frozen estimates change only on explicit refresh; preserve prices, markups and VAT.
