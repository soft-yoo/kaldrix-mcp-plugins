## Preview a manual node or edge edit

1. Read `list_installation_circuits` and the current project `modelVersion`.
2. Submit `preview_installation_circuit_node` with `circuitId`, `expectedModelVersion` and the intended `nodeEdit`.
   It uses the same operation, network fitting, cascades and final geometry guard as saving. No model, history or event is written.
3. Inspect the fitted network, physical segments, quantities and warning codes. MCP warning guidance is English.
4. Save the same intent with `update_installation_circuit` at the same expected version. On `MODEL_VERSION_CONFLICT`,
   read the new state and preview again. Never copy display lane offsets into physical route controls.
5. Previewing `ADD` without explicit new IDs may return temporary IDs; read the actual IDs after saving.
