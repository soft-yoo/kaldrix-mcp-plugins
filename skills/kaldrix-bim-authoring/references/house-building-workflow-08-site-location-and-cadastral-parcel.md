## Site location and cadastral parcel

The site point is separate from BIM geometry: plan outlines use metres, geographic coordinates WGS84 degrees.

1. For a Polish cadastral parcel identifier or district and number, call `find_land_parcel`. With several matches or
   `truncated=true`, ask the user to choose; never silently pick the first.
2. For a chosen map point, call `get_land_parcel_at(latitude, longitude)`: boundary, holes and area. The area is
   calculated from geometry and may differ from the registered cadastral area. Preserve attribution.
3. After user review, save with `set_project_location(projectId, latitude, longitude, ..., confirm=true)`
   (`project-settings.write` and `project-settings.delete`). Optional address text fields are patches: omitted =
   preserve, empty = clear. Map anchors and north rotation stay; the building does not move in its local plan. No new
   tenant-owned collection or file is created.
4. Verify `project.address` with `get_project_detail`, then the saved point with `get_land_parcel_at`. A parcel
   centroid can lie outside a concave parcel or in a hole; if the returned parcel differs, ask for a point inside the
   intended parcel instead of inventing coordinates. The parcel is read from the site point, not saved as an
   ownership or cadastral assignment.

`providerUnavailable=true` means the registry did not respond; an empty response then never proves that no parcel
exists. Lookups share the app's per-user limits and cache. Never send geographic parcel rings to
`set_building_outline` or `set_floor_outline` (plan metres).
