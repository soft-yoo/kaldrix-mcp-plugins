## Playbook: Woda i kanalizacja

1. Presets for kitchens and bathrooms (`add_installation_point_preset` per room or `add_installation_point_presets_for_floor`).
2. Sources with `add_installation_points`: `WATER_MANIFOLD` (else `WATER_RISER` / `WATER_METER`) for water, `DRAIN_STACK`
   for drains (near the bathroom), `HEAT_SOURCE` / `UFH_MANIFOLD` for heating, `GAS_METER` for gas.
3. `suggest_installation_circuits` with `system: "WATER"` then `"DRAINAGE"` → cold + hot water per fixture to the
   manifold, drains to the nearest stack with a 2 % fall (WC PVC-U 110, others 50).
4. `apply_installation_circuit_suggestions` per system; check warnings `DRAIN_DEPTH_EXCEEDED` / `POINT_UNREACHABLE`
   in `list_installation_circuits` and fix with `update_installation_circuit` (`levelOffset`, `slope`).
5. Estimate: `CIRCUIT_LENGTH` filtered by spec + medium (e.g. `[PEX_AL_PEX_16, COLD_WATER, HOT_WATER]`, `[PVC_U_110]`),
   `CIRCUIT_FITTING_COUNT` (`ELBOW`, `TEE`, `END_CAP`), points with `INSTALLATION_POINT_COUNT`.
