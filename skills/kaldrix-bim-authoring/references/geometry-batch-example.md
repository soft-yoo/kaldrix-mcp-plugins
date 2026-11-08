# Batch editing

`apply_geometry_batch` accepts a typed `operations` object and edits one floor atomically. Its bounded groups mirror individually available tools: `labelRooms` (`label_room`, names/functions only), `updateWalls` (`update_wall`, without appearance), `updateSlabs` (`update_slab`, name/type/thickness/3D surface), `createPartitions` (`create_partition`, internal path/type/thickness), `createSlabs` (`create_slab`, standalone 3D surfaces/plain holes), and `removals`.

Room labels and removals require `projects.delete` and `confirm=true`; wall/slab updates require `confirm=true`. Add-only batches remain available with `projects.write` and no confirmation. Ask the user to approve the described changes before setting this flag.

Read the current tool schema for each entry. Example arguments:

```json
{
  "projectId": "<project-uuid>",
  "floorId": "<floor-uuid>",
  "expectedModelVersion": 12,
  "confirm": true,
  "operations": {
    "updateWalls": [{"wallId": "<wall-uuid>", "path": [{"x": 5, "y": 0.24}, {"x": 5, "y": 8.76}]}],
    "labelRooms": [{"spaceId": "<room-uuid>", "name": "Kitchen", "type": "KITCHEN"}],
    "createPartitions": [{"type": "PARTITION", "thickness": 0.12, "path": [{"x": 2, "y": 1}, {"x": 2, "y": 4}]}]
  },
  "idempotencyKey": "reviewed-floor-edit-1"
}
```

`operationsJson` is no longer accepted. Raw installation routes, room dimensions and appearance changes are not batch operations. Use the individually exposed `update_installation_point` and `update_structural_member` for those elements; they are outside this batch. Unknown fields are rejected.

Removals accept only WALL, SLAB, INSTALLATION_POINT, STRUCTURAL_MEMBER, FRAMING_SYSTEM and INSTALLATION_CIRCUIT, corresponding to their registered removal tools. They require `projects.delete` and `confirm=true` after the user approves the complete effect. Removing a wall also removes its openings and attached routes; removing a roof slab removes its roof assembly. Read the tool descriptions for cascades. A stale `expectedModelVersion` or invalid operation rejects the whole batch.

Use outline tools for building footprints. After mutation, re-read `get_floor_detail` for derived room IDs and the current modelVersion. Label derived rooms with `label_room`. Retry with the same input and idempotency key; do not substitute a fresh model version into a previously reviewed edit.
