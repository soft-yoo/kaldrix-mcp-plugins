## Editing and ownership

Use `set_building_outline` or `set_floor_outline` for envelope geometry. Use `update_wall` for an internal partition or permitted envelope properties. Use `remove_wall` with confirmation and delete scope to remove an internal partition; room division is recalculated. Rooms retain matching identities and descriptions where possible. Re-read IDs after splits and merges.

There are no room creation/removal, room merge/split, wall joining, boundary repair or legacy blueprint-build tools. Physical shared partitions exist once. A room label does not move walls, installations or slabs. A room cannot be deleted as a physical element.

Use current `modelVersion` on every versioned mutation. Resolve a version conflict by reading the model and planning again. Inspect operation results for the current version, created IDs and validation feedback.
