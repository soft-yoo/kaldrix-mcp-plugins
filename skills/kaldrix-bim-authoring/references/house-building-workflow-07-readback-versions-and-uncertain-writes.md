## Readback, versions and uncertain writes

Use `get_floor_detail` to obtain the actual envelope axes before drawing partitions. The outer-face footprint is not a wall axis; do not add half-thickness offsets by guessing. Re-read after each physical change and after automatic building placement.

Every physical model write requires `expectedModelVersion` from the state reviewed by the agent. A preview is advisory, not a reserved transaction: after any intervening edit, read the relevant scope and review the changed input before writing. Never substitute a fresh version into a stale proposal. Always inspect the saved result and validation warnings.

Every physical write requires `expectedModelVersion` from the state you reviewed. Every individual element create/add tool also requires a stable `idempotencyKey`. Reuse that key and the exact same input (including the reviewed version) after an uncertain response: the backend returns the original committed result without creating another element. Changed input requires a new key and a fresh reviewed version. Do not identify a retry by name. A confirmed version conflict means reread and replan, not substitute a newer version blindly.

Roof creation/removal, opening intents, installation point edits/presets and framing writes require `expectedModelVersion` from the reviewed model. `preview_framing_system` returns the version to use for its save. Never replace a stale version with a fresh one without reviewing and replanning.

After a replay, the returned result retains its original modelVersion. Read get_project_detail before planning another edit; the live project may have changed since that command committed.
