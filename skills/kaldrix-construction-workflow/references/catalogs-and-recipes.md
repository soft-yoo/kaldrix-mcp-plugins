## Element kinds, resources and reusable recipes

Kaldrix BIM-lite uses element kinds (`ElementType`): floors, spaces, walls, openings, slabs and slab openings,
installation points, circuits and risers, structural members, framing systems, build-ups, construction systems
(footings, ring beams, lintels, stairs, chimney…) and measurements. `list_elements` / `get_element` read any kind;
`get_floor_detail` lists a floor's rooms, walls, slabs and measurements. There is no Revit-style family/type registry.
A roof is modeled from the top storey outline (`create_roof`).

Two independent switches decide what an agent may change. Company modules (INSTALLATIONS, STRUCTURES, ESTIMATES…)
return `MODULE_DISABLED` when off. Project parts (`activeParts` in `get_project_detail`) are chosen per project; a change
to an element of a part that is off returns `PROJECT_PART_DISABLED` with `context.part`. An unknown ID returns
`FLOOR_NOT_FOUND` / `ELEMENT_NOT_FOUND` with `nextTools` that list valid IDs.

Company catalog resources describe materials, labour or equipment. Services (recipes) combine those resources into
repeatable work with norms per unit. A price list supplies resource prices. A project estimate freezes the selected
service, quantities and prices: changing a catalog price is not proof that an existing estimate changed.
