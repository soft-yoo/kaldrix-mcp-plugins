## Outline model contract


A project contains independent buildings. Each building has a stable UUID, a name, a description, a type and an outer-face `baseOutline`. Floors belong to one building and inherit this footprint unless they have an `outlineOverride`. Coordinates use project plan metres (x, y); 3D plan coordinates are (x, z), with height on y.


Use the canonical sequence in `kaldrix://projects/house-building-workflow`: project → building → outline → floors → partitions → room labels.

# Floor-plan Image Analysis

Read and follow this checklist before converting a scan, screenshot, photograph or raster floor plan into BIM geometry.
The MCP backend does not inspect pixels. The client or agent must perform this visual analysis before calling the outline and partition tools.

## 1. Establish the evidence

- Prefer evidence in this order: exact CAD, scaled vector/PDF, dimensioned raster, undimensioned raster, freehand sketch.
- Correct page rotation and perspective mentally or with the client's image tools before tracing.
- Identify the drawing scale from dimension strings. Cross-check at least two independent dimensions when possible.
- Separate published facts from inferred geometry. Keep room areas, overall dimensions, wall thicknesses and levels as
  constraints, not as substitutes for the plan topology.
- If scale, a wall route or an opening is ambiguous, record the ambiguity. Do not invent precision.

### Normalize and read text before tracing

- When client capabilities allow it, crop the drawing, correct rotation and perspective, improve contrast and inspect
  enlarged regions before running OCR or visually reading text.
- Run OCR or equivalent visual text recognition for room names, room areas, dimension strings, floor titles, levels and
  construction notes. Keep the original visible text, the interpreted value and its confidence separate.
- OCR is evidence, not geometry. A label can name a room and a dimension can establish scale, but neither defines a wall
  or room boundary without a matching structural line.
- Cross-check every geometry-critical OCR value against a second dimension, a repeated scale, a published metric or the
  traced geometry. Never turn an uncertain digit into an exact dimension silently.
- If a geometry-critical label or dimension remains unreadable after reasonable inspection, ask the user for the value,
  a clearer image or a marked-up plan before claiming an exact reconstruction.

## 2. Denoise semantically

Treat these as non-geometry unless another reliable signal confirms them:

- furniture, kitchen cabinets, appliances and sanitary fixtures;
- room numbers, names, area labels, dimension strings and leader lines;
- floor hatching, tiles, textures, shadows, reflections and colour fills;
- landscaping, grids, crop borders, section/elevation markers and scan artefacts;
- door swing arcs and window symbols by themselves. They are evidence for an opening, not wall segments.

Preserve continuous wall bands, wall centre lines, wall corners, junctions, clear opening gaps and dimension anchors.

## 3. Recover the exterior envelope first

1. Trace the outside face or a consistent reference line around the whole building as one closed loop.
2. Preserve real recesses, projections and attached volumes. Do not simplify them into detached room rectangles.
3. Check that the loop has no self-intersections and matches published overall dimensions within the evidence tolerance.
4. Save this outer-face polygon with `set_building_outline`; use `set_floor_outline` only for a storey that differs.

One building floor should normally have one connected exterior envelope. A second closed component means either another
building, an intentional detached structure or a tracing error.

## 4. Recover walls and room topology

- Follow continuous wall bands through furniture and labels; use repeated thickness and aligned junctions as evidence.
- Distinguish external, load-bearing and partition walls only when thickness or source notation supports it.
- Snap nearly collinear segments and resolve every corner, L-junction, T-junction and crossing deliberately.
- Build an adjacency graph: list which rooms touch and on which boundary segment before creating polygons.
- Adjacent rooms must reuse one physical wall. Let the backend resolve T-junctions; never create overlapping wall
  copies for each room.
- A partial partition drawn inside one room does not have to split that room. Trace it as a contained standalone wall:
  its `containingSpaceId` identifies the room, it stays out of `boundedByWallIds`, and both of its faces belong to that
  room's interior-finish quantities. If one endpoint reaches a same-type boundary wall, preserve both wall objects and
  create one shared T-junction with `create_partition`.
- Compare the derived clear room outlines with the drawing. Room labels and published areas constrain the review but do
  not define geometry.
- Detect rooms from closed usable areas bounded by structural walls, not from text labels alone. A room without a readable
  label must not disappear from the model.
- If a room outline is reliable but its name or purpose is missing, retain the derived room and record its missing label.
  If the user accepts an approximate model, label it as a neutral `ROOM` named `Room 1`, `Room 2`, and so on, and record the
  missing name/type as an ambiguity. Do not invent `KITCHEN`, `BATHROOM`, `TECHNICAL` or another specialized type.
- If a missing or weak wall changes how many rooms exist, their outlines or their adjacency, stop and ask the user. Do not
  close the wall, merge rooms or create an exact-looking room based only on convenience.
- Rooms plus physical wall-width zones must fill the exterior envelope. Any unexplained enclosed empty area is an
  error. Represent a real courtyard, shaft, atrium, stair opening or service void explicitly.

## 5. Detect doors

Use multiple signals where possible:

- a clear gap in a wall band;
- a door leaf line and swing arc;
- a threshold, opening width or door label;
- circulation evidence connecting two rooms or a room to the exterior.

Record the host wall, connected spaces or exterior side, opening centre, width, height and swing/hinge orientation when
the source supports them. The leaf and swing arc are not walls. Do not turn a furniture edge or missing scan pixels into
a door. An internal room that has no circulation path should be treated as a topology warning requiring review.

## 6. Detect windows

Look for a wall interruption with thin parallel frame/glazing lines, a repeated window symbol or an opening dimension.
Record the host exterior wall, centre position, width, height and sill height when known. Do not confuse wardrobes,
radiators, counters or facade hatching with windows. A window does not create room adjacency.

## 7. Cross-check before encoding

- Exterior width and length agree with source dimensions.
- The sum and individual values of traced room areas agree with published areas within the stated tolerance.
- Every room polygon is closed, non-overlapping and connected to the building layout.
- Every shared room boundary is represented once and every opening has one valid host wall.
- No unexplained area remains inside the effective building footprint.
- Roof outline, terraces, balconies, steps and other external slabs are kept separate from room polygons.

## 8. Record uncertainty and author physical geometry

Keep observed labels, scale, dimensions and unresolved ambiguities in the analysis visible to the user.
Do not send retired blueprint or image-analysis DTOs: these are not active MCP inputs.
Create the project and stable building metadata, set the outer-face building outline, then create floors belonging
 to that building. Use floor overrides only when the drawing shows a different footprint.
Trace internal wall axes once with `create_partition`. Inspect the derived rooms and apply names with `label_room`.
Attach doors and windows to one physical wall with `add_opening_by_intent`.
Configure the existing shared floor and ceiling surfaces; do not create a slab for each room.
Use `validate_floor_geometry`, `validate_project_geometry` and `render_floor_svg` to compare the completed model
with the source. Keep the reference image visible alongside the BIM preview when the client supports it.

## 9. Resolve critical ambiguities

When scale, the exterior envelope, a partition route, floor assignment or an opening host is unresolved,
ask a focused question before encoding that part. Label inferred values visibly. Review room areas and adjacency
after partitions have generated the rooms. Never create independent room polygons as a substitute for physical walls.
