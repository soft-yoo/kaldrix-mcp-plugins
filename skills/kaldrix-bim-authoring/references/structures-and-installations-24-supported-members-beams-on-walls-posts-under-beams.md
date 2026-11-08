## Supported members (beams on walls, posts under beams)

A member end can rest on a support: `WALL`, another `MEMBER`, the `FLOOR` (y = 0) or a `SLAB` (slab top).
`list_structures` shows `startSupport` / `endSupport` (`kind`, `elementId`, `bearing`, `offset` 0..1 along the
support, `level`). The backend re-solves supported ends after **every** change of the support, whichever tool
made it: moving a wall apart lengthens the beam, a thicker wall moves the bearing, a taller wall raises a
`WALL_TOP` beam, a raised beam lifts the post head. Removing a support frees the end in place
(`supportLost = true`, validation INFO `STRUCTURAL_MEMBER_SUPPORT_LOST`). Members that support each other in a
loop are left unchanged (WARNING `STRUCTURAL_MEMBER_SUPPORT_CYCLE`).

1. Beam between two walls — `add_beam_between_walls` with both wall ids and one plan point near each wall
   (projected onto the wall axis; the axis runs through both projections). Each end bears `bearing` metres into
   the wall from its inner face (default min(0.15, thickness)). Height `level`: `WALL_TOP` (underside on the
   wall top, default), `UNDER_CEILING` (top at the floor height) or `CUSTOM` with `elevation` (underside).

```json
{"startWallId":"<wall A>","startPoint":{"x":0.1,"y":2.0},"endWallId":"<wall B>","endPoint":{"x":3.9,"y":2.0},
 "width":0.12,"height":0.24,"role":"BEAM","level":"WALL_TOP"}
```

2. Post under the beam — `add_post_under_member` with the beam `memberId` and `planPoint` (or `offset`), base
   `FLOOR` (default) or `SLAB` + `slabId`. The post stands vertical from the base to the beam underside.
3. Moving a wall later needs no member update — read the new length with `list_structures`; an estimate built
   on `STRUCTURAL_MEMBER_LENGTH` shows the change in `review_estimate_against_model`.

Errors: `STRUCTURAL_SUPPORT_INVALID` (same wall twice, virtual wall, bad level, bearing above the wall
thickness, planPoint and offset both or neither, post base outside the slab), `STRUCTURAL_SUPPORT_UNSOLVABLE`
(beam almost parallel to a wall), `STRUCTURAL_SUPPORT_SPAN_TOO_SHORT` (clear span below 0.3 m, beam underside
not above the post base), `STRUCTURAL_SUPPORT_CYCLE`.
