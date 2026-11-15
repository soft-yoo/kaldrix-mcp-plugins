## Preview a manual node or edge edit

1. Read `list_installation_circuits` and the current project `modelVersion`.
2. `preview_installation_circuit_node` with `circuitId`, `expectedModelVersion` and the intended `nodeEdit` runs the
   same operation, network fitting, cascades and final geometry guard as saving; writes no model, history or event.
3. Inspect the fitted network, physical segments, quantities and warning codes (English guidance).
4. Save the same intent with `update_installation_circuit` at the same version; on `MODEL_VERSION_CONFLICT` read and
   preview again. Never copy display lane offsets into physical route controls.
5. Previewing `ADD` without explicit new IDs may return temporary IDs; read actual IDs after saving.
