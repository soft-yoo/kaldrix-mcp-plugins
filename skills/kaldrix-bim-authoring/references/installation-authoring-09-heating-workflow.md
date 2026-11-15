## Heating workflow

Read `get_floor_modeling_workflow` with `area=INSTALLATIONS` and the current model version.
- `heating` scenario: radiators and their individual supply/return connections.
- `underfloor`: a HEAT_SOURCE or HEATING_RISER, a UFH_MANIFOLD, its paired feed and saved parametric room loops. A room loop alone does not complete the manifold feed or any disconnected radiator.

Use `suggest_installation_circuits` for feed/radiator connections, then preview and create underfloor loops separately. Never apply the paired multiplier to UNDERFLOOR_HEATING quantities. When pricing, send the target estimateId to quantity sources so only its building contributes.

A completed SOURCE step offers SELECT_POINT actions for existing source points on the current floor (also for readers): inspect the returned pointId in `list_installation_points`; the UI opens it without writing. These are not a missing prerequisite or a request to create another source; use currentStep and write actions for the next authoring change.
