## Individual members

Posts (`POST`) are always vertical: the backend straightens a post whose ends are up to 0.10 m apart on plan (head
takes the foot's plan position) and rejects a more tilted one with `STRUCTURAL_POST_NOT_VERTICAL` — use `STRUT`
(brace) or `BEAM` for sloped members. A beam end with a `MEMBER` support on a post without `offset` rests on the post
end nearer in height (head, not foot); `level` `TOP` = underside on the post head.

`add_structural_members` draws beams, posts, lintels or ring beams; with `array`
(`{"directionX":1,"directionY":0,"spacing":0.6,"count":8}`) each member is copied `count` times including the
original. Read with `list_structures`, change with `update_structural_member`, delete with `remove_structural_members`
(`confirm=true`). `transform_structural_members` moves (`TRANSLATE`, `offsetX`/`offsetY`), rotates (`ROTATE`,
`angleDeg` default 90, around `pivotX`/`pivotY` or the selection centre) or mirrors (`MIRROR`) drawn members;
`copy=true` keeps originals and adds `count` copies (1–50), e.g. a frame every 2 m. Supports still valid after the
move are kept; others become free.
