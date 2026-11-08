## Permissions: you act as the person

Every Kaldrix MCP connection acts on behalf of one person in one company chosen on the consent screen. If several
eligible companies are listed, the person selects one before granting access. Switching companies in the app does
not change an existing connector's company; reconnect to choose another company. There is no separate "AI agent" permission:
an active person in a company with the AI module and AI connections enabled may connect an agent. The agent gets only supported connector operations
within that person's access, narrowed by consent and enabled modules; some app operations are never exposed to MCP.

- **Where permissions come from.** Each person in a company gets one access template (for example Administrator,
  Site manager, Worker, Guest, or the company's own templates). Templates belong to the company: they start as the four
  Kaldrix patterns and the company may edit, add or delete them. A template gives areas (projects, estimates,
  invoices, documents, team …) at a level: view (`<area>.read`), changes (`<area>.write`) or full access
  (`<area>.delete`). The fixed template "No access" ("Brak dostępu") blocks access to every site, including sites
  previously assigned to or shared with the person; people land there when their template is deleted. Baseline account
  scopes may remain, so this is not a promise of an empty tool list. Those scopes do not grant access to site data.
  The company owner always has every permission. For other templates, site access (all sites or selected sites)
  further limits project data.
- **Template names are not authorization.** Administrator, Site manager, Worker and Guest are editable template
  labels, not fixed roles. Read actual person scopes, project visibility and connector consent. Even an owner's
  connector cannot bypass missing consent, an excluded module or an unavailable tool.
- **What this connection gets.** Its scopes are the person's current permissions, narrowed to what the person approved
  on the consent screen. They are re-checked on every call: a permission taken away stops working immediately; a newly
  given permission needs the connector to be reconnected and the new scope approved.
- **Never available to connectors**, whatever the person may do in the app: account and sign-in security, team and
  permission management, company settings, subscription and payments, and the KSeF connection. Some removals (for
  example deleting a whole project) exist only in the app.
- **When a call is refused** (`MISSING_SCOPE` with `requiredScope`, `FORBIDDEN`, `WORKSPACE_ACCESS_DENIED`, a
  `*_PERMISSION_REQUIRED` code or a site that is not shared with the person): do not retry, do not look for another tool
  that does the same, and do not ask the user for passwords or tokens. Tell the user in plain words which area or site
  is missing and what to do: someone who manages the team can change their template in Team → Permissions (or give
  access to the site); if they already have the permission in the app, reconnect the connector and approve that scope.
- **Project versions.** Listing and previewing need `history.read`, saving a version `history.write` (both with
  `projects.read`), restoring `history.restore` (with `projects.write`; restoring estimates also `estimates.write`).
  The consent screen lists `history.restore` with removals, unchecked by default.
- **Activity log.** Every tool call of this connection is recorded in the AI activity history (AI → AI activity
  history, "Historia pracy AI"). Each person sees their own agents' actions; people with the Team permission
  (`members.read`) see the whole company.

Saved-data updates to contracts, warranties, crew assignments, time entries and tasks
require both write and delete scopes for their area, plus `confirm=true` after the
user reviews the change. Invoice due-date and stage updates require `invoices.write`
and `invoices.delete`; site-location updates require `project-settings.write` and
`project-settings.delete`. A write-only connection must be authorized again for these
scopes within the person's current permissions. First assignments through these
flagged tools require the same confirmation as replacements.
`confirm=true` is a Boolean submitted by the agent, not proof of a separate human click. Obtain the user's
actual decision before setting it; never infer approval from the flag, an error or a retry.

`list_tool_profiles` shows which tools this connection can use now. Inside a connection, guides and catalogs
(including `kaldrix://mcp/service-catalog`) list only the tools this connection can use; a tool they mention but
you cannot use appears under "Not available in this connection" (Markdown) or `unavailableInThisConnection`
(JSON) with the reason and the missing scope, company module or company type.

Read `list_tool_profiles.unavailableToolGroups` for missing tools and the precise next step.
`MODULE_NOT_IN_PLAN` requires plan review; `MODULE_DISABLED` means an included feature or
dependency is switched off. The owner manages modules; person permissions and OAuth scopes
remain separate. Diagnostics expose metadata only and do not enable hidden tools.
`MODULE_UNAVAILABLE` means the exact module state could not be classified: refresh the catalog and inspect
the current entitlement and feature state before recommending a switch or plan change.
