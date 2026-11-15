## Editing and ownership

- Envelope geometry: `set_building_outline` or `set_floor_outline`. Internal partition or permitted envelope properties: `update_wall`. Remove an internal partition with `remove_wall` (confirmation, delete scope); room division is recalculated, rooms keep matching identities and descriptions where possible. Re-read IDs after splits and merges.
- There are no room creation/removal, room merge/split, wall joining, boundary repair or legacy blueprint-build tools. Shared partitions exist once. A room label does not move walls, installations or slabs; a room cannot be deleted as a physical element.
- Send the current `modelVersion` on every versioned mutation; on a version conflict read the model and plan again. Inspect results for the current version, created IDs and validation feedback.
