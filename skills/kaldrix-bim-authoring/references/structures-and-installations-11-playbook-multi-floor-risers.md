## Playbook: Multi-floor risers

1. `create_installation_riser` with project `planX`/`planY` (shaft, same on every floor), `fromFloorId`/`toFloorId` of
   one building and lines, e.g. `[{"medium":"DRAIN"},{"medium":"COLD_WATER"},{"medium":"HOT_WATER"}]` (drain stack +
   water risers) or `[{"medium":"FEEDER"}]` (WLZ main board → floor boards). An existing `DRAIN_STACK` / `WATER_RISER`
   within 0.6 m is adopted, keeping its circuits. Only media with `riserAllowed: true` in
   `get_installation_conduit_catalog`; underfloor heating loops stay on a floor and cannot be riser lines.
2. Each floor routes to the riser point (`connections[].pointId`): drains fall toward `DRAIN_STACK`; water / heating /
   gas use the riser point as source.
   - FEEDER: create by hand with `create_installation_circuit` (main board → `ELECTRICAL_RISER` on the lowest floor,
     `ELECTRICAL_RISER` → `DISTRIBUTION_BOARD` above). Targets only boards and electrical risers
     (`get_installation_conduit_catalog.media[].targetKinds`); sockets/lights → `INSTALLATION_CIRCUIT_TARGET_INVALID`.
   - Heating targets: radiators, an underfloor heating manifold or a heating riser. Connect the heat source to the riser
     on the lowest floor, then route upper-floor radiators from that riser point. Another heat source as target →
     `INSTALLATION_CIRCUIT_HEATING_TARGET_INVALID`.
   - On the source floor `suggest_installation_circuits` adds `RISER_FEED` proposals: water meter → cold-water riser,
     domestic hot-water source → hot-water riser, heat source → heating riser, gas meter → gas riser. Preview/apply as
     ordinary floor circuits; the configured riser-line spec is reused and saved manual feeds are preserved. FEEDER and
     circulation circuits are not created automatically.
3. The estimate counts vertical metres once (floor-to-floor height per line, runKind `RISER`, e.g. "Pion
   kanalizacyjny — montaż (mb)", "Pion wodny — montaż") and `SLAB_PASS` per line per slab; floor routes stay in the
   circuits — nothing counted twice. `list_installation_risers` warnings: `RISER_NOT_CONNECTED`, `RISER_POINT_MISSING`.
4. Riser points move only via `update_installation_riser` (else `INSTALLATION_POINT_MANAGED_BY_RISER`).
