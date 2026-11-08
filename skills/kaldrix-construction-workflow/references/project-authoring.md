## Project Authoring step by step

Prefer `create_quick_house` for a new house. For a custom model use the relevant topic from the installed `kaldrix-bim-authoring` skill's building workflow index. Without that skill, read `kaldrix://projects/house-building-workflow` on demand. Follow this sequence:

1. `create_project` or inspect an existing project (`get_projects`, `get_project_detail`).
2. Reuse `created.buildingId` returned by `create_project`.
3. `set_building_outline` with the outer-face polygon in metres, unless `create_project` already saved initial dimensions.
4. `create_building_floor` explicitly belonging to that building.
5. Read `get_floor_detail`. The floor outline describes outer wall faces; each wall summary includes exact `axis` points and `envelopeEdgeIndex`.
6. `create_partition` once per physical internal wall axis, connected to existing wall axes; keep every segment within the floor outline.
7. Inspect generated rooms and describe them with `label_room`.
8. `add_opening_by_intent` for windows, doors and cut-outs.
9. `create_roof` on that building's highest floor.

Do not create independent room polygons or duplicate shared walls. Edit an envelope through its outline. Send the reviewed
`modelVersion` as `expectedModelVersion` to physical writes; re-read after intervening edits.
