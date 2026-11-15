## Verification Protocol

Run `validate_project_geometry` for the whole model; resolve `mustFixCodes` until `readyToReport` is true. Render each
floor with `render_floor_svg` and compare outline, rooms and openings with the request. With `readinessScope=GEOMETRY`,
`readyToReport` covers blocking geometry diagnostics only — not authoring completeness, access to every room, stairs,
code compliance or estimate readiness. Review remaining WARNING/INFO diagnostics against the requested stage and report
unresolved omissions. For an estimate, finish with `get_estimate_checks` and a read-back of totals; also inspect
`review_estimate_against_model` applicability issues even when quantities and totals are unchanged.
