## Playbook: Multi-floor risers

1. `create_installation_riser` with project `planX`/`planY` (the shaft, same on every floor), `fromFloorId`/`toFloorId` from the same building and
   lines, e.g. `[{"medium":"DRAIN"},{"medium":"COLD_WATER"},{"medium":"HOT_WATER"}]` (drain stack + water risers) or
   `[{"medium":"FEEDER"}]` (WLZ main board → floor boards). An existing `DRAIN_STACK` / `WATER_RISER` within 0.6 m is
   adopted, so its circuits stay. Choose only media with `riserAllowed: true` in `get_installation_conduit_catalog`;
   underfloor heating loops stay on a floor and cannot be riser lines.
2. Each floor routes to the riser point (`connections[].pointId`): drains fall toward `DRAIN_STACK`, water / heating /
   gas use the riser point as source, FEEDER circuits connect the boards (main board → `ELECTRICAL_RISER` on the lowest
   floor, `ELECTRICAL_RISER` → `DISTRIBUTION_BOARD` above) — create those two by hand with `create_installation_circuit`.
   FEEDER targets are restricted to boards and electrical risers (`get_installation_conduit_catalog.media[].targetKinds`);
   sockets and lights are rejected with `INSTALLATION_CIRCUIT_TARGET_INVALID`.
   Heating targets are radiators, an underfloor heating manifold or a heating riser. Connect the heat source to the
   riser on the lowest floor, then route upper-floor radiators from that riser point. The catalogue exposes these
   target kinds; another heat source as target is rejected with `INSTALLATION_CIRCUIT_HEATING_TARGET_INVALID`.
   On the source floor, `suggest_installation_circuits` includes `RISER_FEED` proposals from a water meter to the cold-water
   riser, a domestic hot-water source to the hot-water riser, a heat source to the heating riser and a gas meter to the gas
   riser. Preview and apply these as ordinary floor circuits. The configured riser-line spec is reused; saved manual feeds
   are preserved. This does not automatically create FEEDER or circulation circuits.
3. The estimate counts the vertical metres once (floor-to-floor height per line, runKind `RISER`, e.g. "Pion
   kanalizacyjny — montaż (mb)", "Pion wodny — montaż") and `SLAB_PASS` per line per slab; floor routes stay in the
   circuits — nothing is counted twice. `list_installation_risers` warnings: `RISER_NOT_CONNECTED`, `RISER_POINT_MISSING`.
4. Riser points move only with `update_installation_riser` (`INSTALLATION_POINT_MANAGED_BY_RISER` otherwise).
