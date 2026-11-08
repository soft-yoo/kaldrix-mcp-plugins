## Site location and cadastral parcel

The site point is separate from BIM geometry. Plan outlines use metres; geographic coordinates use WGS84 degrees.

1. If the user gives a Polish cadastral parcel identifier or district and number, call `find_land_parcel`. Ask the user
   to choose when there are several matches or `truncated=true`; do not silently pick the first match.
2. For a chosen map point, call `get_land_parcel_at(latitude, longitude)`. The backend returns the boundary, holes and
   area. The area is calculated from the geometry and may differ from the registered cadastral area. Preserve attribution.
3. Review the site-point change with the user, then save it with `set_project_location(projectId, latitude, longitude, ..., confirm=true)`.
   The connection needs `project-settings.write` and `project-settings.delete`. Optional address text
   fields are patches: omitted = preserve, empty = clear. Existing map anchors and north rotation remain unchanged;
   the building does not move in its local plan. No new tenant-owned collection or file is created.
4. Read `get_project_detail` to verify `project.address`, then verify the saved point with `get_land_parcel_at`. A parcel
   centroid can lie outside a concave parcel or in a hole; if the returned parcel differs from the selected one, ask
   for a point inside the intended parcel rather than inventing coordinates. The parcel is read from the site point,
   not saved as an ownership or cadastral assignment.

`providerUnavailable=true` means the registry did not respond. An empty response in that case never proves that no
parcel exists. Lookups use the same per-user limits and cache as the app. Do not send geographic parcel rings to
`set_building_outline` or `set_floor_outline`: those expect plan metres.
