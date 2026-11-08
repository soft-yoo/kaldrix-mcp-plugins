## Circuits: cables to the board, pipes to the manifold, drains to the stack

Points are outlets; circuits are the routes between them and their source. The backend routes every circuit
(along the wall faces, through doors, vertical drops at every point; drains fall 2% toward the stack) and the
estimate measures metres per cable / pipe kind, protection devices and fittings.

1. Make sure the floor has the source points: `DISTRIBUTION_BOARD` (electrical), `WATER_MANIFOLD` /
   `WATER_RISER` / `WATER_METER` (water), `DRAIN_STACK` (drainage), `HEAT_SOURCE` / `UFH_MANIFOLD` (heating),
   `GAS_METER` (gas) — add them with `add_installation_points` like any other point.
2. `suggest_installation_circuits` (read-only, optional `system`) groups the free points per Polish practice:
   lighting ≤ 12 points (YDYp 3×1,5, B10), sockets per room ≤ 10 (3×2,5, B16), 400 V hob on its own 5×2,5 circuit,
   one circuit per appliance point (`OVEN`, `DISHWASHER`, `WASHING_MACHINE`, `DRYER`, `WATER_HEATER` 3×2,5 B16;
   `EV_CHARGER` 5×6 B32, 3-phase),
   LAN / TV home runs, cold + hot water per fixture to the manifold, drains to the nearest stack (WC Ø110,
   others Ø50), radiators to the boiler, gas points to the meter. `notes` name a missing source.
3. `apply_installation_circuit_suggestions` saves exactly those circuits in one change. Or build one yourself with
   `create_installation_circuit` (`medium`, `sourcePointId`, `pointIds`; spec / protection / topology default from
   the medium). Cable / pipe kinds come from `get_installation_conduit_catalog`: pick `spec` from
   `media[].allowedSpecs` of the medium (e.g. `YDYP_3X1_5` only for `LIGHTING`, `FLEX_75` only for `VENT_SUPPLY` /
   `VENT_EXTRACT`, `INSULATED_DUCT_160` / `EPP_150` for `VENT_INTAKE` / `VENT_EXHAUST`); `usages` hint at branch / main /
   riser. `media[].routingFields` lists available CircuitRoutingInput fields with inclusive `minimum` / `maximum`,
   model `unit` (`m` or `ratio`, 0.02 = 2%), `defaultValue` at the medium `defaultLevel`, and `manualNetwork`
   applicability. Use these server bounds; `get_installation_circuit_layout` returns the current effective `levelOffset`, `faceOffset`, `slope` and `reserveFactor` after a level change.
   Drainage exposes slope; other media omit it. A manual node network exposes reserveFactor; DRAIN also exposes slope and levelOffset (available depth).
   Any other spec is rejected with `INSTALLATION_CIRCUIT_SPEC_INVALID` (riser lines:
   `INSTALLATION_RISER_SPEC_INVALID`) and the message lists the allowed ones.
4. `list_installation_circuits` / `get_installation_circuit_layout` show `routeLength`, `totalLength` (with the
   reserve; radiator supply/return × 2, underfloor loops once), fittings and warnings (`SOURCE_MISSING`, `POINT_UNREACHABLE`, `DRAIN_DEPTH_EXCEEDED`…).
   Fix with `update_installation_circuit` (`addPointIds`, `removePointIds`, `routingLevel`, `levelOffset`, `faceOffset`, `slope`, `reserveFactor`);
   delete with `remove_installation_circuits` (`confirm=true`, `projects.delete`).
   Removing a member from a circuit leaves the physical point on the floor. To remove the point itself use
   `remove_installation_points`; the backend removes its circuit memberships and branch waypoints too.
   `sourcePointId` changes the source; `clearSource=true` leaves the circuit without a source.
   `routeOverrides: [{targetPointId, waypoints:[{x,y}]}]` adjusts individual branches in plan metres.
   Each entry replaces its branch; empty waypoints restores that branch, `clearRouteOverrides=true` restores
   the whole automatic route. Creation also accepts initial `routeOverrides` and `faceOffset`.
   Read `get_installation_circuit_layout` afterwards: `fromPointId` → `toPointId` identifies actual connections,
   `runs` is the number of parallel conduits. Crossing a source symbol does not create a connection.
   For fully manual line editing use `explode_installation_circuit` with `confirm=true` (`projects.delete`).
   It removes the circuit and creates plain lines, preserving points. The result lists new `installationIds`;
   the routes are then plain installation routes edited in the app (no MCP route tools). CIRCUIT_* estimate sources are lost,
   so estimate these lines with INSTALLATION_LENGTH. A saved change has no undo; going back means restoring a saved project version (`preview_project_version`, then `restore_project_version` after the user's approval). Read `get_installation_circuit_layout` and review the loss of circuit estimate sources before confirming conversion. The optional project history only records committed change summaries.
5. Estimate: services on `INSTALLATION_CIRCUIT` with `CIRCUIT_LENGTH` (MB, filter by spec + medium, e.g.
   `[PEX_AL_PEX_16, COLD_WATER, HOT_WATER]`), `CIRCUIT_COUNT` (PCS, e.g. `[B16]`) and `CIRCUIT_FITTING_COUNT`
   (PCS, at least one of `ELBOW`, `TEE`, `BOX`, `END_CAP`). Point services price the outlet itself, circuit services
   the route — never both for the same material.
6. Electrical rules (indicative PN-HD 60364 warnings, not a certified design): sockets, bathroom and outdoor circuits
   need `rcd = RCD_30MA_A` (the default for new socket circuits and suggestions), ≤ 10 sockets per circuit, the
   protection must fit the cable (3×1,5 → B10/B13, 3×2,5 → B16), a 400 V socket and every appliance point get their own
   circuit (dishwasher, washer, dryer and EV charger with RCD 30 mA), and a
   floor with electrical points needs a `DISTRIBUTION_BOARD`. `validate_floor_geometry` reports them
   (`INSTALLATION_CIRCUIT_RCD_MISSING`, `…_CABLE_TOO_THIN`, `…_DEDICATED_REQUIRED`, `…_THREE_PHASE_CABLE`,
   `INSTALLATION_DISTRIBUTION_BOARD_MISSING`) with actions `SET_CIRCUIT_RCD`, `SET_CIRCUIT_PROTECTION`,
   `SPLIT_DEDICATED_CIRCUIT`. `get_distribution_board` lists the board schedule (number, protection, RCD, cable, phase,
   length), the RCD groups and a module estimate; price RCDs with `CIRCUIT_RCD_COUNT` (PCS — one per RCD group).
