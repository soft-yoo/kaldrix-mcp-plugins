---
name: kaldrix-service-estimation
description: Use when creating or changing Kaldrix service recipes, price lists, estimate positions, model quantity scopes, markups, VAT, estimate offers or model reports. For a read-only estimate review use kaldrix-estimate-review; for an entire house quote start with kaldrix-construction-workflow.
---

# Estimate with the backend's current rules

Use current MCP schemas and backend results for prices, quantities, units,
currency, VAT, totals, scope, readiness and available actions. Never reproduce
the pricing engine or infer a hidden amount. Inspect the selected estimate,
its status, project/building scope and the person's cost visibility first.

Use the [service estimation topic index](references/service-flow.md) to choose
only the fragment for the current task: catalogue/recipe setup, quantity mapping,
estimate editing, model scope proposals, offer lifecycle, PDF or reports. Read
another fragment only when reaching that step or when returned data requires it.
The index assembles into the complete server guide; reading all fragments is
unnecessary for a single position or price-list change.

For where an input comes from or which tool comes next, read
[tool order and inputs](references/tool-order-and-inputs.md).

Backend-generated categories, markup rules, quantity rules and live tool schemas
remain server contracts. Fetch the relevant compact resource when needed rather
than relying on a copied enum or writing another calculation. Keep returned
project, estimate, recipe and position identities distinct. Preserve the
selected building scope and source provenance, reviewed versions and retry keys.

Explain preview changes before applying them. An approved estimate is frozen;
follow the backend's supported change workflow. An editable draft takes model
quantity changes by itself after each model change; read
[the estimate follows the model](references/service-flow-30-estimate-follows-the-model.md)
before applying model changes yourself. Sending an offer, approving it
or recording a payment requires the user's actual decision and any confirmation
declared by the tool. A request to review does not authorize a write.
Replacing saved estimate settings, rows, scopes, prices, recipe composition or
generated PDFs also requires explicit confirmation in the current schemas.
Scope replacement or a manual quantity/formula requires estimates.delete and
confirm=true; append-only scope assignment stays on estimates.write.
`set_estimate_discount` and `update_estimation_item` require both `estimates.write`
and `estimates.delete`; `update_catalog_resource` requires `catalog.write` and
`catalog.delete`. Review the replacement or clearing of saved data with the user,
then set `confirm=true`, including for the first save through these tools.
Read back saved changes and the backend checks; distinguish warnings, missing
scope and a failed check from successful readiness.

Without this skill, read `kaldrix://estimation/service-flow` through
`resources/read` or `describe_kaldrix_mcp(resourceUri=URI)` on demand. Use the full
fallback if the installed reference conflicts with current schemas or lacks a
topic. Never automatically fetch the complete guide before every estimate write.
