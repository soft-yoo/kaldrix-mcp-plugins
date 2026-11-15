## Playbook: Instalacja elektryczna

1. `add_installation_point_presets_for_floor` (sockets, switches, light points per room type); adjust with
   `update_installation_point` / `remove_installation_points`.
2. `add_installation_points`: one `DISTRIBUTION_BOARD` (e.g. hall, wall-mounted).
3. `suggest_installation_circuits` `system: "ELECTRICAL"` → show circuits (lighting ≤ 12 points B10 3×1,5, sockets ≤ 10
   per room B16 3×2,5, 400 V hob 5×2,5, LAN / TV home runs). `missingSources` names the missing source kind; with
   `medium`, existing trade sources lack that line: add the medium to a suitable riser (`update_installation_riser`) or
   add a compatible source point. Sources are chosen per medium (separate supply/extract or cold/hot risers work). A
   local MVHR unit feeds risers even when it also serves local diffusers.
4. `apply_installation_circuit_suggestions` with the same `system`.
5. `list_installation_circuits` → metres per cable (`totalLength` with reserve), fittings, warnings; fix with
   `update_installation_circuit`.
6. Estimate: `INSTALLATION_POINT_COUNT` per point kind + `CIRCUIT_LENGTH` per cable spec + `CIRCUIT_COUNT` per
   protection (B10/B16) + `CIRCUIT_FITTING_COUNT` (`BOX`).
