## Verification Protocol

Run `validate_project_geometry` for the complete model; resolve `mustFixCodes` until `readyToReport` is true. Render
each floor with `render_floor_svg` and compare its outline, rooms and openings with the request. `readinessScope=GEOMETRY` means that `readyToReport` covers blocking geometry diagnostics only. It does not certify authoring completeness, access to every room, stairs, code compliance or estimate readiness. Review remaining WARNING/INFO diagnostics against the requested stage and report unresolved omissions. For an estimate, finish
with `get_estimate_checks` and a read-back of totals; also inspect `review_estimate_against_model` applicability issues even when quantities and totals are unchanged.
