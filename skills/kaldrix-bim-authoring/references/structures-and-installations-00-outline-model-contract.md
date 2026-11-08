## Outline model contract

Every physical write requires the reviewed `expectedModelVersion`. Individual element creation also requires `idempotencyKey`: use a unique stable key for one intent and reuse it only with identical input after an uncertain response. Example keys below are placeholders for independent intents; do not reuse them across real jobs. After a replay read the live project before another edit.



A project contains independent buildings. Each building has a stable UUID, a name, a description, a type and an outer-face `baseOutline`. Floors belong to one building and inherit this footprint unless they have an `outlineOverride`. Coordinates use project plan metres (x, y); 3D plan coordinates are (x, z), with height on y.


Use the canonical sequence in `kaldrix://projects/house-building-workflow`: project → building → outline → floors → partitions → room labels.

# Structures and Installations — playbooks

Installation points (sockets, switches, light points, water supplies, drains, radiators, gas and ventilation points),
installation circuits (routed cables and pipes), structural members (beams, posts, rafters, battens, ring beams),
parametric framing systems (suspended ceilings, timber joists, post-and-girt frames, roof framing) and layered build-ups
(timber-frame walls, masonry, floors, roofs) are separate model elements on a floor. The backend validates them, snaps
points to walls, routes circuits and generates every framing member, fixing and the bill of materials. You send intent;
never compute members, routes or quantities yourself.

Company modules: simple installation points, plain installation lines/groups and their dedicated reads need **INSTALLATIONS**;
circuits, risers, their routed networks, planning guides, HVAC and automation need **INSTALLATION_DESIGN**, which depends
on INSTALLATIONS. Structural authoring and previews, including roof framing (`ROOF_RAFTERS`, roof members),
frame layouts (`get_wall_frame`, `get_slab_frame`), suspended ceilings and timber joists need **STRUCTURES**.
Basic roof geometry, roof takeoff and build-ups (`list_build_ups`, `create_build_up_from_template`, `assign_build_up`…)
remain in **PROJECTS**. Saved model inspection through `list_elements` / `get_element` and saved framing reads
does not grant specialist authoring access; project visibility and read scopes still apply. Estimate tools need
**ESTIMATES**. See [plan and module access](plan-and-module-access.md) for current and legacy entitlements.
On a module refusal, inspect `list_tool_profiles.unavailableToolGroups`: the owner can enable an included module,
but an excluded module needs plan review. Do not retry the refusal or bypass it with a generic batch.

Project parts are chosen per project (`get_project_detail` → `activeParts`, `set_project_parts`, catalog:
`get_project_part_catalog`). `create_project` without `parts` starts with the defaults of the building type, as the
app does: a house has no STRUCTURES, SUSPENDED_CEILINGS or TERRAIN until they are turned on. Creating or changing an element of a part that is off in this project (e.g. installation
points with INSTALLATIONS off, a roof with ROOF off) fails with `PROJECT_PART_DISABLED` and `context.part`. Ask the user
before turning the part on with `set_project_parts`; turning a part off never deletes its elements.

For all-trade manual examples, component choices and supported limits, read
`kaldrix://projects/installation-authoring` before designing water circulation, heating, gas or custom 3D networks.
