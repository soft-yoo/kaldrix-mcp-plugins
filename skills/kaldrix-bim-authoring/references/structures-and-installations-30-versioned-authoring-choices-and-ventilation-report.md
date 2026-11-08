## Versioned authoring choices and ventilation report

Use `get_installation_circuit_selection` with the current `expectedModelVersion` before drafting or
editing a circuit. The backend returns compatible source IDs, media, eligible receiver IDs, and a
typical receiver subset. For point placement or reconnection, send either `targetPointId` or
`targetPointKind` to obtain eligible circuit IDs. The two fields are mutually exclusive; point-kind
selection does not place a point or validate its future position. Existing same-medium assignments
are excluded from other circuits. Refresh the project after a model version conflict.

Use `get_ventilation_route_report` to read room supply/extract airflow, diffuser counts, and actual
flexible duct multiplicities. Counts come from connected routed segments and retain explicit manual
network runs. An unconnected diffuser contributes airflow, but no invented duct. Connected rigid Spiro routes also produce an empty ducts list here; use list_installation_circuits and get_installation_circuit_layout for their lengths and warnings. These tools are
read-only and require INSTALLATION_DESIGN (with its INSTALLATIONS dependency) and projects.read.
If that design module is unavailable, inspect saved elements with the permitted raw model reads;
being able to read saved data does not grant a new planning or design preview.

### Structural member authoring catalog

Call `get_structural_member_catalog` before offering suggested sections or default role materials. The catalog contains role defaults, section dimensions in metres, profile codes and explicit role-specific material overrides. Submit the selected material explicitly to member operations. Custom dimensions and manually chosen materials remain supported; presets do not certify structural capacity.
