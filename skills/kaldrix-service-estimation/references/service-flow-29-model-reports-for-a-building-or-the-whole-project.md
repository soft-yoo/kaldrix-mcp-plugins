
## Model reports for a building or the whole project

`get_project_purchase_report(projectId, buildingId?)` and `get_timber_schedule(projectId, buildingId?)` accept
a building UUID from `get_project_detail`; it excludes other buildings and site geometry. Omit it for the
whole-project report including site. Responses echo `buildingId` (`null` for the whole project); a building
outside the project is rejected with `BUILDING_NOT_FOUND`.

The model purchase report reads current model quantities, not frozen estimate positions; use
`get_estimate_purchase_resources` for a particular saved estimate. Compare raw quantities in the same units.
Timber cut lists are calculated independently per scope, so never add purchased bars or rounded groups from
building reports to rebuild the whole-project cut list.
