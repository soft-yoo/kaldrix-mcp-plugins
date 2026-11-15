## Playbook: Wiata / altana (TIMBER_POST_FRAME)

1. Project with a first floor (`create_project`); the frame needs no rooms.
2. `preview_framing_system` `kind: "TIMBER_POST_FRAME"` with an `outline` of post axes (or a room `spaceId`), e.g.
   `{"outline":[{"x":0,"y":0},{"x":4,"y":0},{"x":4,"y":3},{"x":0,"y":3}],"params":{"mainSpacing":2.0,"postHeight":2.6,"girtCount":1,"openings":[{"edgeIndex":0,"offset":1.5,"width":1.0}]}}`.
   Show posts, girts, braces, plates and the BOM.
3. `create_framing_system` with the same input → `systemId`; `get_framing_drawing` = elevation with the post spacing
   chain. Change via `update_framing_system` (e.g. `girtCount`, `openings`).
4. Roof: set the building outline, create its highest storey, `create_roof` on it (effective outline = footprint; no
   rooms needed), then `ROOF_RAFTERS` for the returned roof assembly. `add_structural_members` + `array` only for
   extra manual members outside the parametric system.
5. Estimate: starter-pack services per role (`FRAMING_ROLE_VOLUME` with `TIMBER_POST_FRAME` + role) plus anchors and
   brackets; no whole-volume position next to them.
