## Five Documents

1. `PriceList` — a company price list. A company can have several (e.g. 2025 and 2026); exactly one is the
   default (`isDefault`): preferred for new estimates with matching project currency and used by catalog
   writes without `priceListId`. The first one is created by `create_price_list`, `copy_price_list`
   (`source=DEMO`), `update_price_list` or the first resource or service write. Fields: name, description,
   `defaultMarkupPercent`, `status` (`ACTIVE`, or `ARCHIVED` = finished and read-only) and a business
   `catalogVersion` that changes only when its resources or service recipes change. Every resource, service
   and estimate belongs to one price list; service components come from the service's price list.
2. `CatalogResource` — a current company material, labor rate or equipment rate. Its price may change.
3. `ServiceRecipe` — a **service** with a billing unit, VAT rate and measurement rule. It has **no price of
   its own**: it groups library resources (its **composition** — components with a calculation mode,
   quantity factor, optional rounding and waste override); its price is the composition cost plus
   `markupPercent`, or the price list's `defaultMarkupPercent` when omitted. Reads return `unitPriceNet` and
   `compositionCostNet`; a service without components costs 0.
4. `ProjectEstimate` — one estimate header under a project. It records the `priceListVersion` its rows were
   frozen from and reports totals of active frozen items.
5. `EstimationItem` — one frozen estimate line with its own quantity, unit, price, VAT and totals, so later
   catalog changes do not rewrite an existing estimate.

Mongo collection names remain unchanged for storage compatibility. Supplier and marketplace semantics are
not part of this flow.

If MCP resources are unavailable, call `describe_kaldrix_mcp` with a `resourceUri` from its input schema,
e.g. `kaldrix://estimation/quantity-rules`, `kaldrix://estimation/catalog-categories` or
`kaldrix://estimation/markup-rules`; it returns the registered reference as text. Do not guess quantity-rule
or enum values from translated UI labels.
