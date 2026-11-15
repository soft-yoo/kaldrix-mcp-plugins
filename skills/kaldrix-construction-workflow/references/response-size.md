## Token Budget

Read only needed resources. Prefer compact navigation before detail reads and follow returned next-tool guidance. Request component or detailed payloads only when needed. A compact page is not the complete estimate.

Paged lists share one contract: send `limit` and, for the next page, `cursor`; read `count`, `hasMore`, `nextCursor` and `total` (when counted). While `hasMore` is true, send `nextCursor` unchanged as `cursor` with the same filters; `nextCursor` is absent on the last page.

Fields without a value are omitted, not null: absent means "not set" or "not applicable". Edit and recipe results carry a compact floor validation: `readyToReport`, counters, `mustFixProblems` and one `suggestedActions` entry per diagnostic code. Run `validate_floor_geometry` for every warning and repair action.

Tool response `_meta` has `kaldrix/callId` for server log correlation and `kaldrix/backendDurationMs` for server processing from session dispatch to a ready response, including authorization and schema validation. It excludes network, gateway and client time, so it cannot explain full connector latency alone.
After a replay the result keeps its original modelVersion; read `get_project_detail` before planning another edit.
