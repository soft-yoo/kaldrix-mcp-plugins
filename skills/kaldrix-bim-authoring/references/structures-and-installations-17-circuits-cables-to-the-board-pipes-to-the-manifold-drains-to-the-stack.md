## Circuits: cables to the board, pipes to the manifold, drains to the stack

Points are outlets; circuits are routes between them and their source. The backend routes every circuit (along wall
faces, through doors, vertical drops at every point; drains fall 2% toward the stack); the estimate measures metres
per cable / pipe kind, protection devices and fittings.

1. Source points on the floor (add via `add_installation_points`): `DISTRIBUTION_BOARD` (electrical), `WATER_MANIFOLD`
   / `WATER_RISER` / `WATER_METER` (water), `DRAIN_STACK` (drainage), `HEAT_SOURCE` / `UFH_MANIFOLD` (heating),
   `GAS_METER` (gas).
2. `suggest_installation_circuits` (read-only, optional `system`) groups free points per Polish practice: lighting
   ≤ 12 points (YDYp 3×1,5, B10), sockets per room ≤ 10 (3×2,5, B16), 400 V hob on its own 5×2,5 circuit, one circuit
   per appliance point (`OVEN`, `DISHWASHER`, `WASHING_MACHINE`, `DRYER`, `WATER_HEATER` 3×2,5 B16; `EV_CHARGER` 5×6
   B32, 3-phase), LAN / TV home runs, cold + hot water per fixture to the manifold, drains to the nearest stack (WC
   Ø110, others Ø50), radiators to the boiler, gas points to the meter. `notes` name a missing source.
3. `apply_installation_circuit_suggestions` saves exactly those circuits in one change. Or `create_installation_circuit`
   (`medium`, `sourcePointId`, `pointIds`; spec / protection / topology default from the medium).
   - `spec` must come from `get_installation_conduit_catalog` `media[].allowedSpecs` (e.g. `YDYP_3X1_5` only for
     `LIGHTING`, `FLEX_75` only for `VENT_SUPPLY` / `VENT_EXTRACT`, `INSULATED_DUCT_160` / `EPP_150` for `VENT_INTAKE` /
     `VENT_EXHAUST`); `usages` hint at branch / main / riser. Other specs → `INSTALLATION_CIRCUIT_SPEC_INVALID` (riser
     lines: `INSTALLATION_RISER_SPEC_INVALID`), message lists allowed ones.
   - `media[].routingFields` lists CircuitRoutingInput fields with inclusive `minimum` / `maximum`, model `unit` (`m`
     or `ratio`, 0.02 = 2%), `defaultValue` at the medium `defaultLevel`, and `manualNetwork` applicability; use these
     server bounds. Drainage exposes slope; other media omit it. A manual node network exposes reserveFactor; DRAIN
     also slope and levelOffset (available depth). `get_installation_circuit_layout` returns the current effective `levelOffset`, `faceOffset`, `slope` and `reserveFactor` after a level change.
4. `list_installation_circuits` / `get_installation_circuit_layout`: `routeLength`, `totalLength` (with reserve;
   radiator supply/return × 2, underfloor loops once), fittings, warnings (`SOURCE_MISSING`, `POINT_UNREACHABLE`,
   `DRAIN_DEPTH_EXCEEDED`…).
   - Fix with `update_installation_circuit` (`addPointIds`, `removePointIds`, `routingLevel`, `levelOffset`,
     `faceOffset`, `slope`, `reserveFactor`); delete with `remove_installation_circuits` (`confirm=true`, `projects.delete`).
   - Removing a member leaves the physical point; `remove_installation_points` removes the point plus its circuit
     memberships and branch waypoints. `sourcePointId` changes the source; `clearSource=true` leaves none.
   - `routeOverrides: [{targetPointId, waypoints:[{x,y}]}]` adjusts branches in plan metres; each entry replaces its
     branch, empty waypoints restore it, `clearRouteOverrides=true` restores the whole automatic route. Creation also
     accepts initial `routeOverrides` and `faceOffset`.
   - Afterwards read `get_installation_circuit_layout`: `fromPointId` → `toPointId` = actual connections, `runs` =
     parallel conduits. Crossing a source symbol creates no connection.
   - Fully manual line editing: `explode_installation_circuit` with `confirm=true` (`projects.delete`) removes the
     circuit and creates plain lines, keeping points; result lists new `installationIds`. These plain routes are
     edited in the app (no MCP route tools). CIRCUIT_* estimate sources are lost — estimate with INSTALLATION_LENGTH.
     No undo: going back means restoring a saved project version (`preview_project_version`, then
     `restore_project_version` after the user's approval). Before confirming read `get_installation_circuit_layout`
     and review the lost circuit estimate sources. Optional project history only records committed change summaries.
5. Estimate: services on `INSTALLATION_CIRCUIT` with `CIRCUIT_LENGTH` (MB, filter by spec + medium, e.g.
   `[PEX_AL_PEX_16, COLD_WATER, HOT_WATER]`), `CIRCUIT_COUNT` (PCS, e.g. `[B16]`), `CIRCUIT_FITTING_COUNT` (PCS, at
   least one of `ELBOW`, `TEE`, `BOX`, `END_CAP`). Point services price the outlet, circuit services the route —
   never both for the same material.
6. Electrical rules (indicative PN-HD 60364 warnings, not a certified design): socket, bathroom and outdoor circuits
   need `rcd = RCD_30MA_A` (default for new socket circuits and suggestions); ≤ 10 sockets per circuit; protection
   must fit the cable (3×1,5 → B10/B13, 3×2,5 → B16); a 400 V socket and every appliance point get their own circuit
   (dishwasher, washer, dryer and EV charger with RCD 30 mA); a floor with electrical points needs a
   `DISTRIBUTION_BOARD`. `validate_floor_geometry` reports `INSTALLATION_CIRCUIT_RCD_MISSING`, `…_CABLE_TOO_THIN`,
   `…_DEDICATED_REQUIRED`, `…_THREE_PHASE_CABLE`, `INSTALLATION_DISTRIBUTION_BOARD_MISSING` with actions
   `SET_CIRCUIT_RCD`, `SET_CIRCUIT_PROTECTION`, `SPLIT_DEDICATED_CIRCUIT`. `get_distribution_board` lists the board
   schedule (number, protection, RCD, cable, phase, length), RCD groups and a module estimate; price RCDs with
   `CIRCUIT_RCD_COUNT` (PCS — one per RCD group).
