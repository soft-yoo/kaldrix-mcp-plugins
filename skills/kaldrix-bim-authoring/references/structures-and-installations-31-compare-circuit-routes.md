## Compare circuit routes

`preview_circuit_optimization` with the current model version and objective `LESS_MATERIAL`, `FEWER_CONNECTIONS`,
`FEWER_WALL_PASSES` or `LOWEST_COST`; explicit branch limit (0–64). Each metric block reports `length` (with reserve,
parallel runs and paired heating), `routeLength`, `fittings` (wall penetrations excluded), `wallPasses` and full physical
`segments` (floor-local x/z plan coordinates, y height in metres). Inspect source/target IDs, both geometries and
trade-offs before applying. `LESS_MATERIAL` never proposes a longer route; other goals may add length only for their
own benefit — say so. Pipe and ventilation variants keep their topology and canonical routing constraints.

The search compares up to three traversal preferences per eligible topology and at most 64 targets: a bounded
heuristic, not a global-optimum proof or standards-compliance certificate. Follow returned `guidance` and `nextTools`:
keep the route on `NO_IMPROVEMENT`, preserve manual networks on `MANUAL_ROUTE`. Full commissioning, equipment sizing
and electrical/hydraulic/thermal calculations are outside this comparison.

Cost comparison: send the project estimate UUID. Pricing uses frozen model-linked conduit, fitting and labor rows,
incl. manual component quantities; missing prices and unknown/manual source quantities give diagnostics, not a
fabricated cost. The estimate is never changed.

Present before/after measurements and cost, get the user’s confirmation, then `apply_circuit_optimization` with the
exact preview input, proposalId and confirm=true. A changed model or frozen-price fingerprint requires another preview.
No undo: going back means restoring a saved project version (`preview_project_version`, then `restore_project_version`
after the user's approval). Project history records committed change summaries.
