## Construction previews and versioned writes

`preview_construction_system` returns `modelVersion` from the same snapshot as its layout (stairs, chimney, railing or managed construction parameters); it saves no system and creates no history. Send that version unchanged to `create_construction_system` or `update_construction_system` with the same intent. After `MODEL_VERSION_CONFLICT` get and review a new preview; never substitute a newer number to force a write.
