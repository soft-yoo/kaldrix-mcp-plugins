## Outline model contract

Every physical write needs the reviewed `expectedModelVersion`; individual element creation also needs `idempotencyKey` (unique and stable per intent; reuse only with identical input after an uncertain response). Example keys below are placeholders, not for reuse across real jobs. After a replay read the live project before another edit.

A project contains independent buildings, each with a stable UUID, name, description, type and outer-face `baseOutline`. Floors belong to one building and inherit this footprint unless they have an `outlineOverride`. Plan coordinates are project metres (x, y); 3D uses (x, z) with height on y. Canonical sequence (`kaldrix://projects/house-building-workflow`): project → building → outline → floors → partitions → room labels.

# Structures and Installations — playbooks

Separate model elements on a floor: installation points (sockets, switches, light points, water supplies, drains,
radiators, gas and ventilation points), installation circuits (routed cables and pipes), structural members (beams,
posts, rafters, battens, ring beams), parametric framing systems (suspended ceilings, timber joists, post-and-girt
frames, roof framing) and layered build-ups (timber-frame walls, masonry, floors, roofs). The backend validates them,
snaps points to walls, routes circuits and generates every framing member, fixing and the bill of materials. You send
intent; never compute members, routes or quantities yourself.

Company modules:
- **INSTALLATIONS**: simple installation points, plain installation lines/groups and their dedicated reads.
- **INSTALLATION_DESIGN** (depends on INSTALLATIONS): circuits, risers, routed networks, planning guides, HVAC, automation.
- **STRUCTURES**: structural authoring and previews, incl. roof framing (`ROOF_RAFTERS`, roof members), frame layouts (`get_wall_frame`, `get_slab_frame`), suspended ceilings, timber joists.
- **PROJECTS**: basic roof geometry, roof takeoff and build-ups (`list_build_ups`, `create_build_up_from_template`, `assign_build_up`…).
- **ESTIMATES**: estimate tools.

Saved model inspection (`list_elements` / `get_element`, saved framing reads) grants no specialist authoring access; project visibility and read scopes still apply. See [plan and module access](plan-and-module-access.md) for current and legacy entitlements. On a module refusal inspect `list_tool_profiles.unavailableToolGroups`: the owner can enable an included module; an excluded module needs plan review. Never retry the refusal or bypass it with a generic batch.

Project parts are per project (`get_project_detail` → `activeParts`, `set_project_parts`, catalog `get_project_part_catalog`). `create_project` without `parts` uses the building type defaults, as the app does: a house has no STRUCTURES, SUSPENDED_CEILINGS or TERRAIN until turned on. Creating/changing an element of a disabled part (e.g. installation points with INSTALLATIONS off, a roof with ROOF off) fails with `PROJECT_PART_DISABLED` and `context.part`. Ask the user before turning the part on with `set_project_parts`; turning a part off never deletes its elements.

Before designing water circulation, heating, gas or custom 3D networks read `kaldrix://projects/installation-authoring` (all-trade manual examples, component choices, supported limits).
