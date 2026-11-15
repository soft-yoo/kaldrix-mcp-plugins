## When a tool is missing

- HTTP 403 for the whole MCP connection with `code=MODULE_DISABLED` and `module=AI`: the AI access gate runs before
  `tools/list` and `list_tool_profiles`, so those diagnostics cannot run. Ask the account owner to check
  **Subscription → Your subscription**; if AI is included in the current access, enable it in
  **Subscription → Features**. Do not retry automatically.
- Otherwise call `list_tool_profiles`: `activeToolNames` (the same names as `tools/list`), `hiddenToolCount`, and
  `unavailableToolGroups` with the hidden names, exact requirement and next step. Diagnostics expose metadata only and
  do not enable hidden tools.
- `MODULE_NOT_IN_PLAN`: the current entitlement excludes the module; the owner reviews
  **Subscription → Compare plans**. A feature switch does not grant an excluded module.
- `MODULE_DISABLED`: the module is included but it or a dependency is switched off; the owner checks
  **Subscription → Features**, then refresh `tools/list`. The owner manages modules; person permissions and OAuth
  scopes stay separate. A tool call refused with `MODULE_DISABLED` (e.g. `allowCopy=true` on a 3D view link without
  PROJECT_COPY_SHARING) names the module but not whether it is outside the plan or switched off; check both.
- `MODULE_UNAVAILABLE`: the module state could not be classified. Refresh the catalog and inspect the current
  entitlement and feature state before recommending a switch or plan change.
- `MISSING_PERMISSION` names the exact missing OAuth scopes, including additional write/delete requirements. A team
  manager checks the person's app permissions; reconnect and approve scopes only after they are granted in the app.
  `COMPANY_TYPE` needs the correct company type.
- A name in `activeToolNames` but missing in the client: refresh its catalog. `UNKNOWN_TOOL`: the name is not
  registered; discover the exact name instead of inventing one.
- Inside a connection, guides and catalogs (including `kaldrix://mcp/service-catalog`) list only usable tools; a
  mentioned but unusable tool appears under **Not available in this connection** (Markdown) or
  `unavailableInThisConnection` (JSON) with the reason and missing scope, company module or company type. Diagnostic
  groups distinguish plan exclusions from feature switches. Another profile, generic batch or differently named tool
  cannot bypass access.
- Never claim a model or estimate was created without a successful tool result.
