## Roof framing edits: support plan

`ROOF_RAFTERS` purlins (`purlins` on/auto) with their posts, and wall plates (`wallPlates`, role `WALL_PLATE`, on the
eaves / high edge of a shed roof) come from a support plan: automatic (`planMode = AUTO`, what the generator would
pick today) or saved and editable (`planMode = CUSTOM`).

Read `get_roof_support_plan(projectId, floorId, systemId)` first. It returns `modelVersion`, `planMode`, the `plan`
currently used (purlins with posts and wall plates, even in AUTO), every post's resolved `postSupports` (bearing
elevation, warning, support candidates, `resolvedKind`/`resolvedElementId` = what the post really stands on: `WALL`,
`MEMBER`, `SLAB`, `FLOOR`, or null for the default level / a support that missed the post), `purlinSupports` (per
built purlin: `canSupportWithBeam`, `beamId`) and generator `warnings`.

Positions (metres, on plan):
- purlin `offsetFromWall` = eave wall face → purlin axis, along the slope; must be > 0; at or beyond the face span the purlin is inactive (warning).
- wall plate `offset` is relative to its own automatic outer-face position (+ towards the building along the slope, - outward), so changing the section does not move where its outer face starts.
- post `position`, purlin `joints`, wall plate `startExtension`/`endExtension`/`joints`: along the eaves from the lowest-coordinate edge of the roof face.

A wall plate with `enabled: false` is not built (rafters left unsupported; warning `WALL_PLATE_DISABLED`). Wall
plates are built only while system parameter `wallPlates` is on, in AUTO and CUSTOM alike: off → none (saved ones stay
stored, unused); on with a saved plan without wall plates → automatic wall plates.

`update_roof_support_plan` (`projectId, floorId, systemId, expectedModelVersion, plan? | materialize? | clear?`)
takes **exactly one** of:
- `plan` — replaces the whole stored plan. `plan.purlins` replaces the whole purlin list (and posts); omitted
  purlins/posts are removed. An explicit `plan.wallPlates` list follows the same rule, **but** omitting it
  (`null`/absent) keeps stored wall plates unchanged (a post/purlin move need not resend them; on an AUTO system the
  automatic wall plates in use are saved), while `[]` stores none, so the roof falls back to automatic wall plates
  while `wallPlates` is on (for a roof without wall plates switch `wallPlates` off with `update_framing_system`).
  Reuse ids from `get_roof_support_plan` to keep editing the same purlin/post/wall plate; invent new stable ids to add one.
- `materialize: true` — saves the current automatic plan (purlins, posts, wall plates) as an editable start (no
  geometry change); saves nothing and stays AUTO only when the automatic plan has neither purlins nor wall plates.
- `clear: true` — **discards** the saved custom plan (all post moves, supports, purlin and wall plate edits lost);
  back to automatic purlin/wall-plate selection. Destructive: ask first; needs `confirm: true` and `projects.delete`.

Typical edit: materialize, then move a post and change what it rests on:

```json
// 1) update_roof_support_plan({..., materialize: true}) → read back plan.purlins[0] (e.g. id "p1" with post "post1")
// 2) update_roof_support_plan({..., plan: {"purlins":[{"id":"p1","faceId":"<faceId from step 1>","offsetFromWall":0.5,
//      "posts":[{"id":"post1","position":2.3,"support":{"kind":"WALL","elementId":"<wall id>","level":"WALL_TOP"}}]}]}})
// plan.wallPlates is omitted here, so the stored wall plates (if any) are kept as they were.
```

Moving `post1` to `position: 2.3` with a wall `support` (instead of automatic resolution) resets every other purlin field to the kind default (width/height, braces `BOTH`, `sill: true`) — send the
whole purlin and post again, since `plan.purlins` replaces the stored list. `update_framing_member` still edits a
single saved generated member's section, rafter eaves/ridge end extension (`startExtension`/`endExtension`), removal
(`removed`) and collar tie height ratio (`collarHeightRatio`); it does not touch the support plan.

Post supports always use the top of the selected wall, beam or slab (floor-local `bottomElevation`); select `kind`
and `elementId`. Generic beam-end `bearing`, `offset` and `level` inputs do not move a post; returned post support
metadata normalizes them to the actual top-bearing behavior. Ordinary structural member end supports keep their own
level and bearing rules.

`support_purlin_with_beam` (`projectId, floorId, systemId, purlinId, expectedModelVersion, width?, height?`) is the
"Podeprzyj podciągiem" shortcut, one atomic change: a timber beam (default 14x24 cm, role `BEAM`) along a built
purlin's axis, ends on the first non-virtual wall on each side (`WALL_TOP`, 0.15 m bearing, like
`add_beam_between_walls`); every post of that purlin landing on the beam gets a `MEMBER` support on it. A system
without a saved plan is materialized first, so it works right after `create_framing_system`.
- No wall on one side → `PURLIN_BEAM_NO_WALL` (add a post there first, or draw the beam with `add_beam_between_walls`).
- All posts already on a beam under it (`purlinSupports[].canSupportWithBeam = false`) → `PURLIN_BEAM_ALREADY_SUPPORTED`; move or remove that beam first.
- Insufficient room for planned posts, or planned posts inactive → whole write rejected with `PURLIN_BEAM_POSTS_DO_NOT_FIT`; no unused beam is saved. An explicitly postless plan still allows adding its beam without inventing posts.
- Result: `beamId` (new member) and `pinnedPostIds` (posts now on it); `add_post_under_member` with that `beamId` adds a post under the beam at a post's position ("Słup pod podciągiem").
- Review support elevations; use an explicitly chosen manual beam level instead of repeating the action.

Set roof and attic geometry and fit the roof to wall plates before detailed support and member edits. After
rebuilding or moving the roof, re-read the plan and member IDs: an earlier beam may now be above a purlin, leaving its
posts inactive; inspect resolved elevations and replan explicitly. Never transfer an inactive member override to
another member by proximity.
