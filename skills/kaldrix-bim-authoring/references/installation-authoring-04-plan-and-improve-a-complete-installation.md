## Plan and improve a complete installation

The agent coordinates intent; the backend owns placement validation, compatible endpoints, topology, physical
routes, fittings and quantities. Never draw guessed straight lines or sum display cable lanes to make an
installation look connected. For every requested trade and floor:

1. Read `plan_floor_installations` at the reviewed version. Inspect existing sources, endpoints and risers; choose
   only requested trades; place missing supported source kinds and room presets. A source serving several trades
   (e.g. gas boiler with domestic hot water) stays one shared device.
2. Preview `suggest_installation_circuits`; review connected point IDs, source repairs, lengths, fittings and
   warnings; apply that exact reviewed version. Diffusers and underfloor have dedicated previews. Read stored
   positions and layouts after saving.
3. Per automatic circuit, `preview_circuit_optimization` with the current reviewed version and goal
   `LESS_MATERIAL`, `FEWER_CONNECTIONS`, `FEWER_WALL_PASSES` or `LOWEST_COST`; `maxBranches` 0–64. The cost goal
   needs an estimate with complete frozen model-derived rows (from the estimate workflow, never an invented price per metre).
4. Inspect `before.segments`/`after.segments`, endpoint IDs, parallel runs, routing preference and quantities
   (physical floor-local paths, not display geometry). Other goals may add material to cut fittings, wall
   penetrations or cost; report the trade-off. Comparison is bounded to eligible topologies and three traversal
   preferences, at most 64 targets; not a global-optimum proof. Plumbing, drainage and ventilation keep their trade topology.
5. If `canApply=true`, keep `proposalId`, model version, objective, branch limit and estimate ID together; after
   confirmation call `apply_circuit_optimization` with exactly those inputs. Version or price-fingerprint conflict →
   reread and replan. `NO_IMPROVEMENT` → keep the route. `MANUAL_ROUTE` → intentionally edit its network with
   `preview_installation_circuit_node`; do not clear it just to enable optimization. Other reason codes: resolve
   their missing inputs/diagnostics first.
6. Re-read trade guides, circuit layouts, riser connections and ventilation report. Run both
   `validate_floor_geometry` and `validate_project_geometry` (unit capacity, cross-floor risers and some
   whole-building checks are outside floor route diagnostics). Review the estimate against the model and explicitly
   accept selected differences; route optimization never changes frozen estimate rows.

Use source/target and routing bounds from the runtime conduit catalogue. Backend checks cover supported electrical
circuit warnings, drainage falls and depth, source compatibility and ventilation route/airflow diagnostics. They do
not establish compliance with electrical, plumbing, gas or ventilation standards, nor provide full
voltage-drop/short-circuit, pressure-loss, heat-load or cross-trade clearance calculations. State which requested
checks are supported and which inputs/calculations remain; never turn `currentStep=null`, empty route warnings or
`canApply=true` into a compliance certificate.
