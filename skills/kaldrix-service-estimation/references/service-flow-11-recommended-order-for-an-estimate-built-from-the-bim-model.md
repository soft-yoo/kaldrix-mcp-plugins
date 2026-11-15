### Recommended order for an estimate built from the BIM model

1. `get_projects` → project UUID; `get_project_detail` lists floors for a floor filter.
2. `get_price_list` and `list_service_recipes` → pick services; `get_service_recipe` per chosen service for
   its `unit`, `quantitySources` and components with `calculationMode = MANUAL` (they need `componentQuantities`).
3. Missing services: confirm the price with the user, then `create_service_recipe` and `add_service_labor_activity`.
4. `list_project_estimates`; `create_project_estimate` when there is none (the first estimate becomes primary).
5. Optional: `set_estimate_sections` (e.g. earthworks, shell, finishing).
6. Whole stages: `price_estimate_from_model` (SHELL, FINISHING, INSTALLATIONS). Each further service:
   `quick_add_estimate_position` → `preview_estimate_position_scope` → `assign_estimate_position_scope` with
   the elements or measurements the user wants, or `update_estimate_sheet` for a hand-given quantity (e.g.
   excavation not in the model). Lower level: `list_estimate_quantity_sources` →
   `add_catalog_service_to_estimate` with `scopes` copied unchanged, plus `sectionId` and `componentQuantities`.
7. Own position outside the price list (transport, a skip, a one-off purchase): `create_estimation_item`.
   `quick_add_estimate_position` needs a `recipeId` or a new service name; with `service-recipes.write` and an
   active price list a new name also adds the service to the price list, otherwise it stays an own estimate row.
8. `get_project_estimate` / `list_estimate_sheet_rows` → report server totals; never sum rows yourself.
9. Before the offer: `get_estimate_checks` (positions without price or quantity, price below cost, model
   stages without positions — each check lists `fixTools`) and `review_estimate_against_model` for changes a
   draft does not take by itself (removed and new elements); then offer details and lifecycle through
   `update_project_estimate` (every call `confirm=true` after the user approves; approval freezes the
   estimate), `generate_estimate_pdf` (`confirm=true`) when the user wants the PDF, and `send_estimate_offer`
   only when the user explicitly asks to send it.

`get_estimate_checks` includes private company cost and profitability checks, so it needs `estimates.read`
and `price-list.read` consent plus the person's live cost access. Without cost access,
`review_estimate_against_model` stays available with `estimates.read` for model quantities, applicability
and coverage, but certifies neither profitability nor full offer readiness. Both share the model-audit
`summary`: a zero projected amount does not clear applicability issues or coverage gaps. Inspect exact
sources before proposing a price or scope change.
