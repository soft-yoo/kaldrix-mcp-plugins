## Compare circuit routes

Use `preview_circuit_optimization` with the current model version and an explicit objective:
`LESS_MATERIAL`, `FEWER_CONNECTIONS`, `FEWER_WALL_PASSES`, or `LOWEST_COST`. Each metric block reports
`length` (with reserve, parallel runs and paired heating), `routeLength`, `fittings` (without wall passes),
`wallPasses` and the full physical `segments` (floor-local x/z plan coordinates, y height in metres). Inspect
source/target IDs, the two geometries and their trade-offs before applying. `LESS_MATERIAL` never
proposes a longer route; other goals may add length only for their own benefit — say so. Pipe and ventilation variants keep
their topology and canonical routing constraints. The branch limit is explicit (0–64).
The search compares up to three traversal preferences per eligible topology and at most 64 targets. It is a
bounded heuristic, not a global-optimum proof or a standards-compliance certificate. Follow returned
`guidance` and `nextTools`: keep the route on `NO_IMPROVEMENT`, and preserve manual networks on `MANUAL_ROUTE`.
Full commissioning, equipment sizing and electrical/hydraulic/thermal calculations are outside this comparison.
For cost comparison send the project estimate UUID. Pricing uses frozen model-linked conduit,
fitting and labor rows, including manual component quantities; missing prices and unknown/manual
source quantities produce diagnostics rather than a fabricated cost. The estimate is never changed.

Present before/after measurements and cost, obtain the user’s confirmation, and call
`apply_circuit_optimization` with the exact preview input, proposalId and confirm=true.
A changed model or frozen-price fingerprint requires another preview. A saved change has no undo; going back means restoring a saved project version (`preview_project_version`, then `restore_project_version` after the user's approval). Project history records committed change summaries.
