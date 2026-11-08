## Units and coordinates (read once)

- Lengths are **in metres**; angles in **degrees** (0 = +X, counter-clockwise). Never centimetres:
  a 14×14 cm post is `width 0.14, height 0.14`.
- Plan X/Y use the shared project coordinate system of room outlines and wall paths (`get_floor_detail`, `get_space_detail`, `get_wall_detail`).
- Installation points: `planX`, `planY`, `height` = metres **above the floor level**.
- Members and generated geometry: `{x, y, z}` with `x`/`z` = plan X/Y and **`y` = height above the floor**
  (a post: `y = 0` → `y = 2.7`; roof members may exceed the floor height).
- Ids: `spaceId`, `wallId` and slab ids from `get_floor_detail` (or `list_elements`), `roofAssemblyId` from
  `create_roof` / `list_structures`, `memberId` / `systemId` from `list_structures`, `pointId` from
  `list_installation_points`, `circuitId` from `list_installation_circuits`, `buildUpId` from `list_build_ups`.
- Every write returns the new ids and `modelVersion`; reads are cheap — re-read instead of guessing.

Reference values: `kaldrix://projects/installation-point-kinds`, `kaldrix://projects/structural-member-roles`,
`kaldrix://projects/framing-system-kinds`, `kaldrix://projects/build-up-templates`, `kaldrix://estimation/quantity-rules`.
