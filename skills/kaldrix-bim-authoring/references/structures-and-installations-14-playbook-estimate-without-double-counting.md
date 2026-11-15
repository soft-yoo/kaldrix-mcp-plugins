## Playbook: Estimate without double counting

1. `install_construction_starter_pack` (idempotent, prices untouched) → `list_service_recipes`.
2. Per element pick **either** the whole-system service ("Więźba dachowa — całość (zamiast osobnych pozycji)",
   "Strop drewniany — całość …") **or** per-role services ("Krokwie — drewno C24", "Murłata", "Łaty 5×4" for roof
   battens) — never both. Same for `WALL_LAYER_VOLUME` `["TIMBER_FRAME"]` vs `WALL_FRAME_ROLE_*`. A batten ceiling
   (`CEILING_TIMBER`, drawn or assumed from a slab build-up) has own grid services in the ceilings trade: "Ruszt sufitu
   z łat — łaty / kontrłaty / kotwy" (`FRAMING_MAIN_LENGTH`, `FRAMING_SECONDARY_LENGTH`, `FRAMING_FIXING_COUNT`);
   suspended ceilings are priced only by the finishing stage. `price_estimate_from_model` (stage SHELL) makes this
   choice itself (per-role services when they cover the whole system, otherwise the whole-system service alone).
3. Per service: `list_estimate_quantity_sources` → `add_catalog_service_to_estimate` with the returned scopes.
4. `review_estimate_against_model` → `doubleCounted` groups: after the user agrees follow `suggestedAction`
   (`REMOVE_ONE` → remove the covering position or the others with `remove_estimate_position`; `NARROW_FILTER` →
   narrow `elementSubtypes` with `update_service_recipe` or drop a scope with `replace_estimate_position`). Intended
   doubles: `set_estimate_double_count_accepted` `accepted=true`.
5. Report totals from `get_project_estimate`; never sum rows yourself.

# Reference
