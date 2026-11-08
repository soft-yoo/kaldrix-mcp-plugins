## Plan and improve a complete installation

The agent coordinates the intent; the backend owns placement validation, compatible endpoints, topology,
physical routes, fittings and quantities. Do not draw guessed straight lines or sum display cable lanes to make
an installation look connected. Work through these steps for every requested trade and floor:

1. Read `plan_floor_installations` at the reviewed project version. Inspect existing sources, endpoints and
   risers; choose only the requested trades. Place missing supported source kinds and room presets. Sources
   that serve several trades (such as a gas boiler with domestic hot water) remain one shared device.
2. Preview `suggest_installation_circuits`, review connected point IDs, source repairs, lengths, fittings and
   warnings, then apply that exact reviewed version. Use the dedicated diffuser and underfloor previews for
   those systems. Read the stored positions and layouts after saving.
3. For each automatic circuit, call `preview_circuit_optimization` with the current reviewed version and a
   chosen goal: `LESS_MATERIAL`, `FEWER_CONNECTIONS`, `FEWER_WALL_PASSES` or `LOWEST_COST`. Set
   `maxBranches` from 0 to 64. The cost goal also needs an estimate with complete frozen model-derived rows;
   obtain it through the estimate workflow, not an invented price per metre.
4. Inspect `before.segments` and `after.segments`, endpoint IDs, parallel runs, routing preference and
   quantities. These are physical floor-local paths, not display geometry. Other goals may increase material
   length to reduce fittings, wall penetrations or cost; report that trade-off. The comparison is bounded to
   the backend's eligible topologies and three traversal preferences, with at most 64 targets. It is not a
   global-optimum proof. Plumbing, drainage and ventilation keep their trade topology.
5. If `canApply=true`, keep `proposalId`, model version, objective, branch limit and estimate ID together.
   After confirmation, call `apply_circuit_optimization` with exactly those preview inputs. Reread and replan
   on a version or price-fingerprint conflict. If `NO_IMPROVEMENT`, keep the current route. If `MANUAL_ROUTE`,
   inspect and intentionally edit its network with `preview_installation_circuit_node`; do not clear it merely
   to enable optimization. Other reason codes require their missing inputs or diagnostics to be resolved first.
6. Re-read the trade guides, circuit layouts, riser connections and ventilation report. Run both
   `validate_floor_geometry` and `validate_project_geometry`: unit capacity, cross-floor risers and some
   whole-building checks are outside one floor's route diagnostics. Review the estimate against the model
   and explicitly accept selected quantity differences; route optimization never changes frozen estimate rows.

Use the source/target and routing bounds from the runtime conduit catalogue. The backend includes supported
electrical circuit warnings, drainage falls and depth checks, source compatibility and ventilation route/airflow
diagnostics. These checks do not establish complete compliance with electrical, plumbing, gas or ventilation
standards. They do not provide full voltage-drop/short-circuit, pressure-loss, heat-load or cross-trade clearance
calculations. State which requested checks are supported and which engineering inputs or calculations remain;
never turn `currentStep=null`, empty route warnings or `canApply=true` into a compliance certificate.
