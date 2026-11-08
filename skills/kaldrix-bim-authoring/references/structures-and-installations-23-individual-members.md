## Individual members

Posts (`POST`) are always vertical: the backend straightens a post whose ends are up to 0.10 m apart on the plan
(the head takes the plan position of the foot) and rejects a more tilted one with `STRUCTURAL_POST_NOT_VERTICAL` —
use `STRUT` (brace) or `BEAM` for sloped members. A beam whose end has a `MEMBER` support on a post without an
`offset` rests on the post end nearer in height (the head, not the foot); `level` `TOP` = underside on the post
head.

`add_structural_members` draws beams, posts, lintels or ring beams. With `array`
(`{"directionX":1,"directionY":0,"spacing":0.6,"count":8}`) each member is copied `count` times including the
original. Read them with `list_structures`; change with `update_structural_member`; delete with
`remove_structural_members` (`confirm=true`). `transform_structural_members` moves (`TRANSLATE` with
`offsetX`/`offsetY`), rotates (`ROTATE`, `angleDeg`, default 90, around `pivotX`/`pivotY` or the selection centre) or
mirrors (`MIRROR`) selected drawn members; `copy=true` keeps the originals and adds `count` copies (1–50), e.g. a frame
copied every 2 m. Supports that still hold after the move are kept, others become free.
