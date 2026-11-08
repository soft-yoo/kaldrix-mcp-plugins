### Estimate without double counting (playbook)

1. `install_construction_starter_pack` (once per price list) → `list_service_recipes` and pick, per element, either the
   whole-system service ("… — całość (zamiast …)") or the per-role services — never both.
2. Per service: `list_estimate_quantity_sources` → `add_catalog_service_to_estimate` with the returned scopes.
3. `review_estimate_against_model` → resolve `doubleCounted` as above, then `changedCount` / `newElements`.
4. Report totals from `get_project_estimate`.

Manual complete-system services need a separate scope check. For example, "Wet underfloor heating — complete
system" is a manually entered `M2` service covering pipe, membrane, clips, edge strip and pressure testing,
excluding manifold and screed. Its aggregated material cost per square metre does not provide pipe metres.
The model-linked underfloor pipe services use `CIRCUIT_LENGTH` in `MB`, including model leads and reserve,
and cover only the selected pipe and its laying. Choose the intended scope; do not add both for the same work.
`doubleCounted` detects overlapping model quantity atoms and shared components. It cannot guarantee detection
of an overlap with an unlinked manual cost basket using different resources or billing units.

Quantities read from the model are clear dimensions: room areas and perimeters are measured to the inner faces of the
walls, not to the outline drawn on the plan, and the wall face of a room is measured between those faces as well.
