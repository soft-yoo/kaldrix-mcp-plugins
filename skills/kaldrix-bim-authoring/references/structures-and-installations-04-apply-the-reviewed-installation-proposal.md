## Apply the reviewed installation proposal

`suggest_installation_circuits` and `suggest_ventilation` return the `modelVersion` of the planned snapshot
(ventilation proposals and whole-project balance belong to it). Both previews validate their canonical BIM operation
without saving.

`expectedModelVersion` is required on `apply_installation_circuit_suggestions` and `apply_ventilation_suggestions`:
send exactly the reviewed preview's `modelVersion`, the same floor and, for circuits, the same `system` and `language`.
An intervening edit returns `MODEL_VERSION_CONFLICT`: read again, refresh the proposal and let the user review it.
Never replace a reviewed version with a newer `get_project_detail` value to bypass the conflict. A successful write
returns actual IDs and the new model version from the same atomic operation; do not guess IDs from an earlier read.
