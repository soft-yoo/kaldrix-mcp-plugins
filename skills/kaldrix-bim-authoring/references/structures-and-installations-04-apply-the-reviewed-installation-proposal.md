## Apply the reviewed installation proposal

`suggest_installation_circuits` and `suggest_ventilation` return `modelVersion` from the snapshot that was planned.
Ventilation proposals and the whole-project balance belong to that same snapshot. Both previews validate their
proposed canonical BIM operation without saving anything.

Send this exact value as `expectedModelVersion` to `apply_installation_circuit_suggestions` or
`apply_ventilation_suggestions`. Keep the same floor and, for circuits, the same `system` and `language`.
An intervening model edit returns `MODEL_VERSION_CONFLICT`; refresh the proposal and let the user review it again.
Never replace a reviewed version with a newer `get_project_detail` value just to bypass the conflict.

`expectedModelVersion` is required on both apply tools: send the `modelVersion` of the reviewed preview. A newer model
is rejected; read again and review a new preview instead of applying an older proposal. A successful write
returns actual IDs and the new model version from the same atomic operation. It does not guess IDs in an earlier read.
