## Roof framing edits: support plan

`ROOF_RAFTERS` purlins (`purlins` on/auto) and their posts, plus wall plates (`wallPlates`, role `WALL_PLATE`,
on the eaves / the high edge of a shed roof), come from a support plan: either the automatically chosen one
(`planMode = AUTO`, same purlins/posts/wall plates the generator would pick today) or a saved, editable one
(`planMode = CUSTOM`). Read `get_roof_support_plan(projectId, floorId, systemId)` first — it returns the
`modelVersion`, `planMode`, the `plan` currently used to build the layout (purlins with their posts and wall
plates, even in AUTO mode), every post's resolved `postSupports` (bearing elevation, warning, support
candidates, and `resolvedKind`/`resolvedElementId` — what the post really stands on: `WALL`, `MEMBER`, `SLAB`,
`FLOOR`, or null for the default level / a support that missed the post), `purlinSupports` (per built purlin:
`canSupportWithBeam`, `beamId`) and generator `warnings`. Positions are in metres, measured on plan: a purlin's `offsetFromWall` is
the distance from the eave wall face to the purlin axis, along the slope; a wall plate's `offset` is relative to
its own automatic outer-face position instead (+ towards the building along the slope, - outward), so moving the
section does not change where its outer face starts; a post `position`, purlin `joints` and a wall plate's
`startExtension`/`endExtension`/`joints` are all measured along the eaves from the lowest-coordinate edge of the
roof face. `offsetFromWall` must be > 0; a value at or beyond the face span leaves the purlin inactive (warning).
A wall plate with `enabled: false` is not built (its rafters are left unsupported; warning
`WALL_PLATE_DISABLED`). Wall plates are built only while the system parameter `wallPlates` is on, in AUTO and
CUSTOM alike: off → no wall plates (the saved ones stay stored, unused); on with a saved plan without wall plates →
the automatic wall plates.

`update_roof_support_plan` (`projectId, floorId, systemId, expectedModelVersion, plan? | materialize? | clear?`)
takes **exactly one** of:
- `plan` — replaces the whole stored plan. `plan.purlins` replaces the whole purlin list (and their posts);
  omitted purlins/posts are removed. `plan.wallPlates` follows the same whole-list-replace rule for an explicit
  list, **but** omitting it (`null`/absent) keeps the stored wall plates unchanged — a plan edit that only moves a
  purlin or a post does not need to resend them (on an AUTO system the automatic wall plates in use are saved) —
  while an empty list `[]` stores none, so the roof falls back to the automatic wall plates while `wallPlates` is
  on (switch `wallPlates` off with `update_framing_system` for a roof without wall plates). Reuse ids
  from `get_roof_support_plan` to keep editing the same purlin/post/wall plate, or invent new stable ids to add
  one.
- `materialize: true` — saves the current automatic plan (purlins, posts and wall plates) as an editable starting
  point (no geometry change); it saves nothing and the roof stays in AUTO only when the automatic plan has neither
  purlins nor wall plates.
- `clear: true` — **discards** the saved custom plan (all post moves, supports, purlin and wall plate edits are
  lost); the roof goes back to automatic purlin/wall-plate selection. Destructive: ask the user first; needs
  `confirm: true` and the `projects.delete` scope.

Typical edit: materialize first, then move a post and change what it rests on.

`support_purlin_with_beam` rejects the entire write with `PURLIN_BEAM_POSTS_DO_NOT_FIT` when the proposed
beam would leave insufficient room for its planned posts or the planned posts are inactive. No unused beam is saved.
An explicitly postless plan still allows adding its beam without inventing posts.
Review the support elevations; use an explicitly chosen manual beam level instead of repeating the action.

Set the roof and attic geometry and fit the roof to wall plates before detailed support and member edits.
After rebuilding or moving the roof, read the current plan and member IDs again. A beam placed earlier may
now be above a purlin, leaving its posts inactive; inspect the resolved elevations and replan explicitly.
Never transfer an inactive member override to another member by proximity.

Post supports always use the top of the selected wall, beam or slab (floor-local `bottomElevation`);
select `kind` and `elementId`. Generic beam-end `bearing`, `offset` and `level` inputs do not move a post.
The returned post support metadata normalizes those fields to the actual top-bearing behavior; ordinary
structural member end supports retain their separate level and bearing rules.

```json
// 1) update_roof_support_plan({..., materialize: true}) → read back plan.purlins[0] (e.g. id "p1" with post "post1")
// 2) update_roof_support_plan({..., plan: {"purlins":[
//      {"id":"p1","faceId":"<faceId from step 1>","offsetFromWall":0.5,
//       "posts":[{"id":"post1","position":2.3,
//                 "support":{"kind":"WALL","elementId":"<wall id>","level":"WALL_TOP"}}]}
//    ]}})
// plan.wallPlates is omitted here, so the stored wall plates (if any) are kept as they were.
```

Moving `post1` to `position: 2.3` and setting its `support` to a wall (instead of the automatic resolution)
keeps every other purlin field at the kind default (width/height, braces `BOTH`, `sill: true`) — send the whole
purlin and post again, since `plan.purlins` replaces the stored purlin list entirely. `update_framing_member`
still edits the section, rafter eaves/ridge end extension (`startExtension`/`endExtension`), removal (`removed`)
and the collar tie height ratio (`collarHeightRatio`) of a single saved generated member; it does not touch the
support plan.

`support_purlin_with_beam` (`projectId, floorId, systemId, purlinId, expectedModelVersion, width?, height?`) is the
"Podeprzyj podciągiem" shortcut: it builds a timber beam (default 14x24 cm, role `BEAM`) along a built purlin's
axis, resting its two ends on the first non-virtual wall found on each side (`WALL_TOP`, 0.15 m bearing, like
`add_beam_between_walls`), and gives every post of that purlin that lands on the beam a `MEMBER` support on it —
one atomic change. A system without a saved plan is materialized first, so this also works right after
`create_framing_system`. No wall on one side of the purlin → `PURLIN_BEAM_NO_WALL` (put a post there first, or
draw the beam by hand with `add_beam_between_walls`). Every post of the purlin already standing on a beam under it
(`purlinSupports[].canSupportWithBeam = false`) → `PURLIN_BEAM_ALREADY_SUPPORTED`; move or remove that beam first. The result carries `beamId` (the new structural member) and
`pinnedPostIds` (the purlin's posts now resting on it); use `add_post_under_member` with that `beamId` for a post
under the beam at a post's position ("Słup pod podciągiem").
