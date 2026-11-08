## Playbook: Wiata / altana (TIMBER_POST_FRAME)

1. A project with a first floor (`create_project`); no rooms are needed for the frame itself.
2. `preview_framing_system` `kind: "TIMBER_POST_FRAME"` with an `outline` of the post axes (or `spaceId` of a room),
   e.g. `{"outline":[{"x":0,"y":0},{"x":4,"y":0},{"x":4,"y":3},{"x":0,"y":3}],"params":{"mainSpacing":2.0,
   "postHeight":2.6,"girtCount":1,"openings":[{"edgeIndex":0,"offset":1.5,"width":1.0}]}}`. Show posts, girts,
   braces, plates and the BOM.
3. `create_framing_system` with the same input → `systemId`; `get_framing_drawing` gives the elevation with the post
   spacing chain. Change it with `update_framing_system` (e.g. `girtCount`, `openings`).
4. Roof: set the building outline and create its highest storey, then call `create_roof` on that storey.
   Its effective outline is the footprint; rooms are not required. Create `ROOF_RAFTERS` for the returned roof assembly.
   Use `add_structural_members` + `array` only for additional manual members outside the parametric system.
5. Estimate: starter-pack services per role (`FRAMING_ROLE_VOLUME` with `TIMBER_POST_FRAME` + role) plus anchors and
   brackets; no whole-volume position next to them.
