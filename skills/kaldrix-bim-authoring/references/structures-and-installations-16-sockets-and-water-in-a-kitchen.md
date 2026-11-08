## Sockets and water in a kitchen

1. `get_floor_detail` → find the kitchen `spaceId` and its outline, and the wall behind the worktop.
2. `add_installation_points` with one call for the whole kitchen (1–200 points, atomic). For wall kinds set
   `wallId` and a plan point on the room side of the wall — the backend projects it onto the wall face and
   derives the facing:

```json
{"points":[
  {"kind":"SOCKET_DOUBLE","planX":1.2,"planY":0.1,"wallId":"<wall>","height":1.1,"label":"Blat"},
  {"kind":"SOCKET_DOUBLE","planX":2.4,"planY":0.1,"wallId":"<wall>","height":1.1,"label":"Blat"},
  {"kind":"POWER_SOCKET_400V","planX":3.0,"planY":0.1,"wallId":"<wall>"},
  {"kind":"WATER_SINK","planX":1.8,"planY":0.1,"wallId":"<wall>"},
  {"kind":"DRAIN_SINK","planX":1.8,"planY":0.1,"wallId":"<wall>"},
  {"kind":"WATER_DISHWASHER","planX":2.0,"planY":0.1,"wallId":"<wall>"},
  {"kind":"LIGHT_POINT_CEILING","planX":2.0,"planY":1.8},
  {"kind":"SWITCH","planX":0.3,"planY":3.5}
]}
```

   Omit `height` for the kind default (sockets 0.30 m, switches 1.10 m, ceiling kinds at the ceiling or the
   room's suspended ceiling). Without `wallId` a wall within 0.5 m is attached automatically.
3. `list_installation_points` to read back the snapped positions and `countsByKind`.
4. Move or change a point with `update_installation_point` (`pointIds` applies the same height, label or kind to
   several points at once); copy points to another storey with `copy_to_floor`; delete with `remove_installation_points`
   (`confirm=true`, `projects.delete`).
