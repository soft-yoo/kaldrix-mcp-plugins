## Playbook: Mechanical ventilation with heat recovery

System rozdzielaczowy (the Polish standard): MVHR unit ↔ air distribution box by a main duct (spiro / EPP Ø160), box →
every diffuser by flexible ducts Ø75 (≈ 30 m³/h each, so a 50 m³/h bathroom gets 2 runs), unit ↔ intake and exhaust by
insulated ducts Ø160. Indicative rules, not a ventilation design.

1. Rooms must exist and be typed (`SpaceType` + names like "Salon", "Sypialnia", "WC", "Pralnia", "Garderoba").
2. `suggest_ventilation` per floor → proposed `SUPPLY_DIFFUSER` (living room 60, bedroom 40, office 30 m³/h) and
   `EXTRACT_DIFFUSER` (kitchen 50 / 70 with gas, bathroom 50, WC 30, laundry 30, pantry 15) plus the project balance.
   `apply_ventilation_suggestions` adds them (corridors are transfer zones — no diffuser).
3. `add_installation_points`: `MVHR_UNIT` (technical room or attic; `airflow` = nominal capacity ≥ `requiredCapacity`),
   `AIR_DISTRIBUTION_BOX` (one per direction, or one two-chamber box), `AIR_INTAKE` and `AIR_EXHAUST` on external walls
   ≥ 1.5 m apart (or exhaust ≥ 1 m higher), optional `SILENCER`.
4. Unit on another floor (attic → ground floor): `create_installation_riser` with lines `VENT_SUPPLY` + `VENT_EXTRACT`
   between the two floors → a `VENT_RISER` point appears on each floor; the planner then routes unit → riser on the
   unit floor and riser → box on the other floor.
5. `suggest_installation_circuits` `system: "VENTILATION"` → `apply_installation_circuit_suggestions`. Read
   `list_installation_circuits` warnings (`FLEX_RUN_TOO_LONG` > 15 m → move / add a box; `VENT_POINT_WRONG_KIND`).
6. `suggest_ventilation` again: balance warnings (`VENT_UNBALANCED` ±10 %, `VENT_UNIT_UNDERSIZED`, `VENT_INTAKE_TOO_CLOSE`,
   `VENT_DIFFUSER_UNCONNECTED`) each say what to change; set airflows with `update_installation_point` `airflow`.
7. Estimate (starter pack): points — "Rekuperator — montaż", "Regulacja i rozruch instalacji wentylacji",
   "Czerpnia / wyrzutnia", "Skrzynka rozdzielcza", "Anemostat / zawór"; routes — `CIRCUIT_LENGTH` per spec (`FLEX_75`,
   `SPIRO_160`, `INSULATED_DUCT_160`, risers included), `CIRCUIT_FITTING_COUNT` `PLENUM` and `ELBOW`/`TEE`/`REDUCER`/
   `COUPLING` with ventilation media, `SLAB_PASS` for slab penetrations.
