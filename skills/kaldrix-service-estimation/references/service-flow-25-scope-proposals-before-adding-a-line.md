## Scope proposals before adding a line

Call `get_estimate_scope_proposals` with projectId, serviceIds and the current expectedModelVersion. An
optional spaceId or elementId (mutually exclusive) narrows the intent. The response has canonical quantity
sources, proposed selectedSourceIds, mode, scopeLocked and availableQuantity. A manual quantity remains user
input, validated by the quantity plan and save operations. Refresh stale model proposals instead of guessing
geometry or quantity rules. Requires the ESTIMATES module and estimates.read scope.
