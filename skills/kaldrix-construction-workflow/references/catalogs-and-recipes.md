## Element kinds, resources and reusable recipes

Kaldrix BIM-lite element kinds (`ElementType`): floors, spaces, walls, openings, slabs and slab openings, installation
points, circuits and risers, structural members, framing systems, build-ups, construction systems (footings, ring
beams, lintels, stairs, chimney…) and measurements. `list_elements` / `get_element` read any kind; `get_floor_detail`
lists a floor's rooms, walls, slabs and measurements. There is no Revit-style family/type registry. A roof is modeled
from the top storey outline (`create_roof`).

Two independent switches limit changes: company modules (INSTALLATIONS, STRUCTURES, ESTIMATES…) return
`MODULE_DISABLED` when off; project parts (`activeParts` in `get_project_detail`) are chosen per project, and changing
an element of a part that is off returns `PROJECT_PART_DISABLED` with `context.part`. An unknown ID returns
`FLOOR_NOT_FOUND` / `ELEMENT_NOT_FOUND` with `nextTools` listing valid IDs.

Company catalog resources are materials, labour or equipment; services (recipes) combine them into repeatable work
with norms per unit; a price list supplies resource prices. A project estimate freezes the selected service,
quantities and prices: a changed catalog price does not prove an existing estimate changed.
