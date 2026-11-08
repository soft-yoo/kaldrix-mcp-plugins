### Recommended order for an estimate built from the BIM model

1. `get_projects` → project UUID; `get_project_detail` lists floors when you need a floor filter.
2. `get_price_list` and `list_service_recipes` → pick services; `get_service_recipe` for each chosen service to read
   its `unit`, `quantitySources` and components with `calculationMode = MANUAL` (they need `componentQuantities`).
3. Missing services: confirm the price with the user, then `create_service_recipe` and `add_service_labor_activity`.
4. `list_project_estimates`; `create_project_estimate` when there is none (the first estimate becomes the primary one).
5. Optional: `set_estimate_sections` to create sections such as earthworks, shell, finishing.
6. Whole stages from the model: `price_estimate_from_model` (SHELL, FINISHING, INSTALLATIONS). For each further service:
   `quick_add_estimate_position` → `preview_estimate_position_scope` → `assign_estimate_position_scope` with the
   elements or measurements the user wants, or `update_estimate_sheet` for a quantity the user gives by hand (e.g.
   excavation that is not in the model). Lower-level tools: `list_estimate_quantity_sources` →
   `add_catalog_service_to_estimate` with `scopes` copied unchanged, plus `sectionId` and `componentQuantities`.
7. An own position outside the price list (transport, a skip, a one-off purchase): `create_estimation_item`.
   `quick_add_estimate_position` needs a `recipeId` or a new service name; with `service-recipes.write` and an active
   price list a new name also adds the service to the price list, otherwise it stays an own estimate row.
8. `get_project_estimate` / `list_estimate_sheet_rows` → report totals from the server; never sum rows yourself.
9. Before the offer goes out: `get_estimate_checks` (positions without a price or quantity, price below cost, model
   stages without positions — each check lists `fixTools`) and `review_estimate_against_model` for model changes
   a draft does not take by itself (removed and new elements), then offer details and the lifecycle through `update_project_estimate` (every call with
   `confirm=true` after the user approves; approval also freezes the estimate), then `generate_estimate_pdf` (`confirm=true`) when the user wants the PDF and `send_estimate_offer` when the user
   explicitly asks to send it.

`get_estimate_checks` includes private company cost and profitability checks, so it requires both
`estimates.read` and `price-list.read` consent plus the person's live cost access. Without cost access,
`review_estimate_against_model` remains available with `estimates.read` for model quantities, applicability
and coverage. It does not certify profitability or full offer readiness. Both results share the model-audit
`summary`: a zero projected amount does not clear applicability issues or coverage gaps. Inspect their exact
sources before proposing any price or scope change.
