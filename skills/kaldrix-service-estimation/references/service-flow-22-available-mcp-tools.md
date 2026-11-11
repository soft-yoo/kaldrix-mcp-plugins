## Available MCP Tools

All tools belong to the single `workspace` catalog; permissions, company modules and company type decide
which of them a connection can use. Lower-level tools give finer control than the simple position flow.

Read:

- `list_price_lists`, `get_price_list`, `list_demo_price_lists`;
- `list_catalog_resources`, `get_catalog_resource`;
- `list_service_recipes`, `get_service_recipe`;
- `list_project_estimates`, `get_project_estimate`;
- `list_estimate_sheet_rows`, `preview_estimate_position_scope`;
- `review_estimate_against_model`, `get_estimate_model_changes`, `get_estimate_checks`, `get_estimate_offer_workflow`;
- lower-level reads: `list_estimation_items`, `preview_manual_estimate_item`, `preview_estimate_budget`,
  `list_estimate_quantity_sources`, `list_element_service_quantities`, `compare_price_lists`.

Write:

- `create_price_list`, `update_price_list`, `copy_price_list` (own price list or demo catalog), `set_default_price_list`,
  `archive_price_list`;
- `import_price_list`, `adjust_price_list_prices`, `copy_service_to_price_list`;
- `change_estimate_price_list`;
- `add_service_labor_activity`, `install_construction_starter_pack`;
- `create_catalog_resource`, `update_catalog_resource`;
- `create_service_recipe`, `update_service_recipe`;
- `create_project_estimate`, `update_project_estimate`, `copy_project_estimate`, `set_primary_project_estimate`;
- `quick_add_estimate_position`, `assign_estimate_position_scope`, `update_estimate_sheet`,
  `price_estimate_from_model`, `apply_estimate_model_changes`, `skip_estimate_new_elements`,
  `acknowledge_estimate_model_changes`, `detach_estimate_items_from_model`;
- `set_estimate_sections`, `assign_estimate_position_to_section`;
- `refresh_estimation_item_prices` (`confirm=true` with `includeOverridden`);
- `generate_estimate_pdf` (stores a new project document);
- `send_estimate_offer` (`estimates.write`) and `revoke_estimate_offer` (`estimates.delete`), both with `confirm=true`;
- lower-level writes: `add_catalog_service_to_estimate`, `replace_estimate_position`, `set_estimate_double_count_accepted`,
  `create_estimation_item`, `update_estimation_item`.

Always with `confirm=true` after the user approves: `update_project_estimate`,
`update_service_recipe`, `update_catalog_resource`, `change_estimate_price_list`, `update_estimate_sheet`,
`apply_estimate_model_changes`, `replace_estimate_position`, `update_estimation_item` and `generate_estimate_pdf`.
Only in some cases: `import_price_list` when it overwrites existing data, `adjust_price_list_prices` unless `dryRun`,
`assign_estimate_position_scope` when it replaces sources, and `set_estimate_sections` when it deletes sections.

Confirmed destructive actions:

- `remove_estimate_position`;
- `remove_estimate_items_with_missing_source` (positions whose model element is gone);
- `deactivate_catalog_resource`;
- `deactivate_service_recipe`;
- `deactivate_project_estimate`;
- `delete_price_list`.
