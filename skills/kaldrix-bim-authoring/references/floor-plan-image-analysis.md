# Floor-plan Image Analysis

Checklist before converting a scan, screenshot, photograph or raster floor plan into BIM geometry. The MCP backend
does not inspect pixels: the client or agent does this visual analysis before calling outline and partition tools.
Model contract and canonical sequence: `kaldrix://projects/house-building-workflow`.

## 1. Establish the evidence

- Evidence priority: exact CAD, scaled vector/PDF, dimensioned raster, undimensioned raster, freehand sketch.
- Correct rotation and perspective before tracing. Take scale from dimension strings; cross-check two independent
  dimensions when possible.
- Room areas, overall dimensions, wall thicknesses and levels are constraints (published facts), not substitutes for
  plan topology (inferred geometry).
- Record ambiguous scale, wall routes or openings; do not invent precision.

### Normalize and read text before tracing

- When possible crop, correct rotation and perspective, raise contrast and enlarge regions before reading text.
- Run OCR or equivalent visual text recognition for room names, areas, dimension strings, floor titles, levels and
  construction notes; keep visible text, interpreted value and confidence separate.
- OCR is evidence, not geometry: without a matching structural line, labels and dimensions define no wall or room
  boundary. Cross-check every geometry-critical value (second dimension, repeated scale, published metric or traced
  geometry); never silently turn an uncertain digit into an exact dimension.
- If a geometry-critical label or dimension stays unreadable, ask for the value, a clearer image or a marked-up plan
  before claiming an exact reconstruction.

## 2. Denoise semantically

Non-geometry unless reliably confirmed: furniture, cabinets, appliances, sanitary fixtures; room numbers, names, area
labels, dimension strings, leader lines; hatching, tiles, textures, shadows, reflections, colour fills; landscaping,
grids, crop borders, section/elevation markers, scan artefacts; door swing arcs and window symbols alone (opening
evidence, not wall segments). Preserve continuous wall bands, centre lines, corners, junctions, clear opening gaps and
dimension anchors.

## 3. Recover the exterior envelope first

1. Trace the outside face (or one consistent reference line) around the whole building as one closed loop.
2. Keep real recesses, projections and attached volumes; never simplify them into detached room rectangles.
3. Check: no self-intersections, overall dimensions match within the evidence tolerance.
4. Save the outer-face polygon with `set_building_outline`; `set_floor_outline` only for a differing storey.

A building floor normally has one connected envelope; a second closed component is another building, an intentional
detached structure or a tracing error.

## 4. Recover walls and room topology

- Follow wall bands through furniture and labels; repeated thickness and aligned junctions are evidence. Classify
  external, load-bearing and partition walls only when thickness or notation supports it.
- Snap nearly collinear segments; resolve every corner, L-junction, T-junction and crossing deliberately. Build an
  adjacency graph (which rooms touch, on which segment) before creating polygons.
- Adjacent rooms must reuse one physical wall. Let the backend resolve T-junctions; never create overlapping wall
  copies for each room.
- A partial partition inside one room need not split it: trace a contained standalone wall (`containingSpaceId` = the
  room, not in `boundedByWallIds`, both faces in that room's interior-finish quantities). If one endpoint reaches a
  same-type boundary wall, keep both walls and create one shared T-junction with `create_partition`.
- Labels and published areas constrain the review of derived clear room outlines but do not define geometry. Detect
  rooms from closed usable areas bounded by structural walls; a room without a readable
  label must not disappear from the model.
- Missing name or purpose on a reliable outline: keep the room, record the missing label. If the user accepts an
  approximate model, use neutral `ROOM` named `Room 1`, `Room 2`, … and record the ambiguity; never invent `KITCHEN`,
  `BATHROOM`, `TECHNICAL` or another specialized type.
- If a missing or weak wall changes room count, outlines or adjacency, stop and ask; never close walls, merge rooms or
  create exact-looking rooms for convenience.
- Rooms plus wall-width zones must fill the envelope. Any unexplained enclosed empty area is an error; represent a real
  courtyard, shaft, atrium, stair opening or service void explicitly.

## 5. Detect doors

Signals: a gap in a wall band; a leaf line and swing arc; a threshold, opening width or door label; circulation
between rooms or to the exterior. Record host wall, connected spaces or exterior side, centre, width, height and
swing/hinge when supported. Leaf and arc are not walls; furniture edges or missing scan pixels are not doors. An
internal room without a circulation path is a topology warning to review.

## 6. Detect windows

Signals: a wall interruption with thin parallel frame/glazing lines, a repeated window symbol or an opening dimension.
Record host exterior wall, centre, width, height and sill height when known. Wardrobes, radiators, counters and facade
hatching are not windows. A window creates no room adjacency.

## 7. Cross-check before encoding

- Exterior width and length match the source; room areas (each and total) match published areas within tolerance.
- Every room polygon is closed, non-overlapping and connected to the layout; each shared boundary exists once; each
  opening has one valid host wall.
- No unexplained area inside the effective building footprint; roof outline, terraces, balconies, steps and other
  external slabs stay separate from rooms.

## 8. Record uncertainty and author physical geometry

Keep observed labels, scale, dimensions and open ambiguities visible to the user. Never send retired blueprint or
image-analysis DTOs (not active MCP inputs). Then:

1. Project and stable building metadata, outer-face building outline, floors of that building (overrides only for a
   different drawn footprint).
2. Internal wall axes once with `create_partition`; name the derived rooms with `label_room`.
3. Doors and windows on one physical wall each with `add_opening_by_intent`.
4. Configure the existing shared floor and ceiling surfaces; no slab per room.
5. Compare with the source via `validate_floor_geometry`, `validate_project_geometry` and `render_floor_svg`; keep
   the reference image beside the BIM preview when the client supports it.

## 9. Resolve critical ambiguities

Unresolved scale, envelope, partition route, floor assignment or opening host: ask a focused question before encoding
that part. Label inferred values visibly. Review room areas and adjacency after partitions generate rooms. Never
create independent room polygons as a substitute for physical walls.
