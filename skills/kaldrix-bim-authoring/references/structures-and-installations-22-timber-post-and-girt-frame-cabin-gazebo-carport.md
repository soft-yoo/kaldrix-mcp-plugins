## Timber post-and-girt frame (cabin, gazebo, carport)

`TIMBER_POST_FRAME` is a SPACE-scope framing system: send a room `spaceId` or an `outline` whose edges are the **post
axes** (edge `i` runs from vertex `i` to `i + 1`, the last edge closes the outline; the layout `outline` returned by
preview / create is the reference for edge indices).

```json
{"kind":"TIMBER_POST_FRAME","name":"Altana","outline":[{"x":0,"y":0},{"x":4,"y":0},{"x":4,"y":3},{"x":0,"y":3}],
 "params":{"mainSpacing":2.0,"postHeight":2.6,"girtCount":1,"braces":true,"openings":[{"edgeIndex":0,"offset":1.5,"width":1.0}]}}
```

Defaults:
- Posts 14×14 (`mainWidth` × `mainHeight`) at every corner, beside every opening and evenly between, spacing ≤ `mainSpacing` (0.5–6 m, default 2.0).
- Sill plate and top plate 14×14 (`sillPlate`, `topPlate`, `plateWidth`, `plateHeight`) along the whole outline.
- `girtCount` rows (0–6, default 1) of girts 12×14 (`secondaryWidth` × `secondaryHeight`) evenly between sill and top plate, face to face of the posts.
- Knee braces 10×10 from every corner post in both directions (`braces`, `braceLength` 0.8 m, `braceAngleDeg` 45, `braceWidth`, `braceHeight`).
- `elevation` = sill underside above the floor (default 0); `postHeight` = sill underside to top-plate top (default: up to storey height).

`openings` leave a door / passage gap: posts on both sides, no girts or braces inside (sill stays continuous).
`offset` / `width` are the **clear opening** between jamb post faces — jamb post axes sit half a post section outside it
(a corner post serves as the jamb at a corner). Openings outside the edge, overlapping or narrower than 0.3 m fail
with `FRAMING_PARAMS_INVALID` and an English MCP message.

Roles (layout and `explode_framing_system`): `POST`, `GIRT` (Rygiel), `STRUT` (Miecz), `BOTTOM_PLATE` (Podwalina),
`TOP_PLATE` (Oczep). BOM: posts `FRAMING_MAIN_LENGTH`; girts and braces `FRAMING_SECONDARY_LENGTH`; plates
`FRAMING_PERIMETER_LENGTH`; sill anchors (or one post base per post without a sill) `FRAMING_FIXING_COUNT`; angle
brackets at post feet / heads, girt and brace ends `FRAMING_CONNECTOR_COUNT`; pieces = members (no stock bars);
`FRAMING_TIMBER_VOLUME` = all frame timber. The construction starter pack prices it per role (`FRAMING_ROLE_VOLUME`
with `TIMBER_POST_FRAME` + role: posts, girts, sill, top plate, braces) plus anchors and brackets — no whole-volume
position next to the per-role ones. `get_framing_drawing` returns one wall elevation (edge nearest the cut origin,
default longest edge) with posts, girts, braces, plates, post spacing chain, height and openings.
