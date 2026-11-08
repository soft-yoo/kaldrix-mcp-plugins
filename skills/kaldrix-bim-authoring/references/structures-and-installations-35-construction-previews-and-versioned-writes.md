## Construction previews and versioned writes

`preview_construction_system` returns `modelVersion` from the same snapshot as its layout (stairs, chimney, railing or managed construction parameters). Send it unchanged to `create_construction_system` or `update_construction_system` with the same intent. After `MODEL_VERSION_CONFLICT`, obtain and review a new preview; never replace the reviewed version with a newer number to force a write. The preview does not save a system or create history.
