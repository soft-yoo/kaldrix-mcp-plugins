## Scope proposals before adding a line

Call `get_estimate_scope_proposals` with projectId, serviceIds, and the current expectedModelVersion.
An optional spaceId or elementId narrows the intent; they are mutually exclusive. The response
contains canonical quantity sources, proposed selectedSourceIds, mode, scopeLocked, and
availableQuantity. A manual quantity remains user input and is validated by the
existing quantity plan and save operations. Refresh stale model proposals rather than guessing
geometry or quantity rules. This read requires the ESTIMATES module and estimates.read scope.
