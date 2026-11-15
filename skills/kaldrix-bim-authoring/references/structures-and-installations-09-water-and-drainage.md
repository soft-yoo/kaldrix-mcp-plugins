## Playbook: Woda i kanalizacja

1. Kitchen/bathroom presets (`add_installation_point_preset` per room or `add_installation_point_presets_for_floor`).
2. Sources via `add_installation_points`: water `WATER_MANIFOLD` (else `WATER_RISER` / `WATER_METER`), drains
   `DRAIN_STACK` (near the bathroom), heating `HEAT_SOURCE` / `UFH_MANIFOLD`, gas `GAS_METER`.
3. `suggest_installation_circuits` `system: "WATER"` then `"DRAINAGE"` → cold + hot per fixture to the manifold,
   drains to the nearest stack with 2 % fall (WC PVC-U 110, others 50).
4. `apply_installation_circuit_suggestions` per system; check `DRAIN_DEPTH_EXCEEDED` / `POINT_UNREACHABLE` in
   `list_installation_circuits`, fix with `update_installation_circuit` (`levelOffset`, `slope`).
5. Estimate: `CIRCUIT_LENGTH` filtered by spec + medium (e.g. `[PEX_AL_PEX_16, COLD_WATER, HOT_WATER]`, `[PVC_U_110]`),
   `CIRCUIT_FITTING_COUNT` (`ELBOW`, `TEE`, `END_CAP`), points via `INSTALLATION_POINT_COUNT`.
