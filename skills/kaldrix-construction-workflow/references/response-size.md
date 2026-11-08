## Token Budget

Read only needed resources. Prefer compact navigation before detail reads and follow returned next-tool guidance. Request component or detailed payloads only when needed. A compact page is not the complete estimate.

Every paged list uses the same contract: send `limit` and, for the next page, `cursor`; read `count`, `hasMore`, `nextCursor` and `total` (present when counted). Send `nextCursor` unchanged as `cursor`, with the same filters, while `hasMore` is true. `nextCursor` is absent on the last page.

Responses omit fields without a value instead of returning null; an absent field means "not set" or "not applicable". Edit and recipe results carry a compact floor validation: `readyToReport`, counters, `mustFixProblems` and one `suggestedActions` entry per diagnostic code. Run `validate_floor_geometry` when you need every warning and repair action.

Tool response `_meta` includes `kaldrix/callId` for server log correlation and `kaldrix/backendDurationMs` for server processing from session dispatch to a ready response, including authorization and schema validation. This timing excludes network, gateway and client time; it cannot explain full connector latency by itself.
After a replay, the returned result retains its original modelVersion; read `get_project_detail` before planning another edit.
