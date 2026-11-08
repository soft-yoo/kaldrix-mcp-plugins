## Across floors

Use `create_installation_riser` with fromFloorId/toFloorId, shaft planX/planY and separate lines for required media. Read actual connection point IDs with `list_installation_risers`. Feed the riser on the source floor and use its compatible connection point on each served floor. Add missing media via `update_installation_riser`; do not attach a hot-water or extract circuit to a riser lacking that line. Vertical takeoff is counted by the backend; do not draw a duplicate vertical plain route.
