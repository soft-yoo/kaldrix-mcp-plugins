### Estimate without double counting (playbook)

1. `install_construction_starter_pack` (once per price list) → `list_service_recipes`; per element pick
   either the whole-system service ("… — całość (zamiast …)") or the per-role services — never both.
2. Per service: `list_estimate_quantity_sources` → `add_catalog_service_to_estimate` with the returned scopes.
3. `review_estimate_against_model` → resolve `doubleCounted` as above, then `changedCount` / `newElements`.
4. Report totals from `get_project_estimate`.

Manual complete-system services need a separate scope check. E.g. "Wet underfloor heating — complete
system" is a manually entered `M2` service covering pipe, membrane, clips, edge strip and pressure testing,
excluding manifold and screed; its aggregated material cost per square metre gives no pipe metres. The
model-linked underfloor pipe services use `CIRCUIT_LENGTH` in `MB` (including model leads and reserve) and
cover only the selected pipe and its laying. Choose the intended scope; never add both for the same work.
`doubleCounted` detects overlapping model quantity atoms and shared components; it cannot guarantee
detection of an overlap with an unlinked manual cost basket using different resources or billing units.

Model quantities are clear dimensions: room areas and perimeters are measured to the inner wall faces, not
to the outline drawn on the plan, and a room's wall face is measured between those faces too.
