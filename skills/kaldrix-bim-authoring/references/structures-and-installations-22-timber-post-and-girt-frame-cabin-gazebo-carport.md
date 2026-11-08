## Timber post-and-girt frame (cabin, gazebo, carport)

`TIMBER_POST_FRAME` is a SPACE-scope framing system: send a room `spaceId` or an `outline` whose edges are the
**post axes** (edge `i` runs from outline vertex `i` to vertex `i + 1`, the last edge closes the outline; the
layout `outline` returned by preview / create is the reference for edge indices).

```json
{"kind":"TIMBER_POST_FRAME","name":"Altana",
 "outline":[{"x":0,"y":0},{"x":4,"y":0},{"x":4,"y":3},{"x":0,"y":3}],
 "params":{"mainSpacing":2.0,"postHeight":2.6,"girtCount":1,"braces":true,
           "openings":[{"edgeIndex":0,"offset":1.5,"width":1.0}]}}
```

Defaults: posts 14×14 (`mainWidth` × `mainHeight`) at every corner, beside every opening and evenly between with
spacing ≤ `mainSpacing` (0.5–6 m, default 2.0); sill plate and top plate 14×14 (`sillPlate`, `topPlate`,
`plateWidth`, `plateHeight`) along the whole outline; `girtCount` rows (0–6, default 1) of girts 12×14
(`secondaryWidth` × `secondaryHeight`) evenly between sill and top plate, face to face of the posts; knee braces
10×10 from every corner post in both directions (`braces`, `braceLength` 0.8 m, `braceAngleDeg` 45, `braceWidth`,
`braceHeight`). `elevation` = underside of the sill above the floor (default 0); `postHeight` = sill underside to
top-plate top (default: up to the storey height). `openings` leave a door / passage gap: posts on both sides, no
girts and no braces inside (the sill stays continuous). `offset` / `width` are the **clear opening** between the jamb
post faces — the jamb post axes sit half a post section outside it (a corner post serves as the jamb at a corner). Openings outside the edge, overlapping or narrower than
0.3 m fail with `FRAMING_PARAMS_INVALID` and an English MCP message.

Roles in the layout and in `explode_framing_system`: `POST`, `GIRT` (Rygiel), `STRUT` (Miecz), `BOTTOM_PLATE`
(Podwalina), `TOP_PLATE` (Oczep). BOM: posts on `FRAMING_MAIN_LENGTH`, girts and braces on
`FRAMING_SECONDARY_LENGTH`, plates on `FRAMING_PERIMETER_LENGTH`, sill anchors (or one post base per post without
a sill) on `FRAMING_FIXING_COUNT`, angle brackets at post feet / heads, girt and brace ends on
`FRAMING_CONNECTOR_COUNT`, pieces = members (no stock bars), `FRAMING_TIMBER_VOLUME` = all frame timber. The
construction starter pack prices it per role (`FRAMING_ROLE_VOLUME` with `TIMBER_POST_FRAME` + role: posts, girts,
sill, top plate, braces) plus anchors and brackets — do not add a whole-volume position next to the per-role ones.
`get_framing_drawing` returns the elevation of one wall (the edge nearest the cut origin, default: longest edge)
with posts, girts, braces, plates, the post spacing chain, the height and the openings.
