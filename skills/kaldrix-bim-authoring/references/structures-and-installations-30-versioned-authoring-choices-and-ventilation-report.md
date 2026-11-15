## Versioned authoring choices and ventilation report

Before drafting or editing a circuit call `get_installation_circuit_selection` with the current `expectedModelVersion`:
compatible source IDs, media, eligible receiver IDs and a typical receiver subset. For point placement or reconnection
send either `targetPointId` or `targetPointKind` (mutually exclusive) to get eligible circuit IDs; point-kind selection
neither places a point nor validates its future position. Existing same-medium assignments are excluded from other
circuits. Refresh the project after a model version conflict.

`get_ventilation_route_report`: room supply/extract airflow, diffuser counts and actual flexible duct multiplicities
(from connected routed segments, keeping explicit manual network runs). An unconnected diffuser contributes airflow
but no invented duct. Connected rigid Spiro routes also give an empty ducts list here; use list_installation_circuits
and get_installation_circuit_layout for their lengths and warnings. These tools are read-only and need
INSTALLATION_DESIGN (with its INSTALLATIONS dependency) and projects.read. Without that module inspect saved elements
via permitted raw model reads; reading saved data grants no new planning or design preview.

### Structural member authoring catalog

Call `get_structural_member_catalog` before offering suggested sections or default role materials: role defaults,
section dimensions in metres, profile codes and explicit role-specific material overrides. Send the selected material
explicitly to member operations. Custom dimensions and manual materials stay supported; presets do not certify
structural capacity.
