## When a tool is missing

- If the whole MCP connection returns HTTP 403 with `code=MODULE_DISABLED` and `module=AI`,
  the AI access gate runs before `tools/list` and `list_tool_profiles`; those diagnostics cannot run.
  Ask the account owner to check **Subscription → Your subscription**. If AI is included in the
  account's current access, enable it in **Subscription → Features**. Do not retry automatically.
- When the MCP connection is available, call `list_tool_profiles`. It reports `activeToolNames` (the same names as `tools/list`),
  `hiddenToolCount`, and `unavailableToolGroups` with the hidden names, exact requirement and next step.
- `MODULE_NOT_IN_PLAN` means the account's current entitlement excludes the module. Ask the owner
  to review **Subscription → Compare plans**. A feature switch does not grant an excluded module.
- `MODULE_DISABLED` means the module is included but it or a dependency was switched off.
  Ask the owner to check **Subscription → Features**, then refresh `tools/list`.
  A tool call refused with error code `MODULE_DISABLED` (for example `allowCopy=true` on a 3D view link without
  PROJECT_COPY_SHARING) names the module but not whether it is outside the plan or switched off; check both.
- `MODULE_UNAVAILABLE` does not establish whether a module is outside the plan or switched off.
  Refresh the catalog and inspect the current entitlement and feature state before recommending either change.
- `MISSING_PERMISSION` names the exact missing OAuth scopes, including any additional write/delete
  requirement. A team manager checks the person's app permissions; reconnect and approve scopes
  only after they are granted in the app. `COMPANY_TYPE` needs the correct company type.
- A name in `activeToolNames` but absent in the client means its catalog needs refreshing.
  `UNKNOWN_TOOL` means the name is not registered: discover the exact name instead of inventing one.
- Guides and catalogs mark hidden references as **Not available in this connection** or
  `unavailableInThisConnection`. Use the diagnostic groups to distinguish plan exclusions from
  feature switches. Another profile, generic batch or differently named tool cannot bypass access.
- Never claim a model or estimate was created without a successful tool result.
