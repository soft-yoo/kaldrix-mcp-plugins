## Across floors

`create_installation_riser` takes fromFloorId/toFloorId, shaft planX/planY and separate lines for required media; read actual connection point IDs with `list_installation_risers`. Feed the riser on the source floor and use its compatible connection point on each served floor. Add missing media via `update_installation_riser`; never attach a hot-water or extract circuit to a riser lacking that line. Backend counts vertical takeoff; do not draw a duplicate vertical plain route.
