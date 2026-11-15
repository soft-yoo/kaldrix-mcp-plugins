## Manual item and budget previews

Call `preview_manual_estimate_item` before `create_estimation_item` when the user supplies a custom service
or a separate material, labor or equipment cost. Send the same project, estimate, name, kind, quantity, unit,
price, VAT, optional resource, section and material waste to both (RESOURCES: purchase price as
`purchasePriceNet`; otherwise sale price as `unitPriceNet`). The preview writes nothing and uses the write
path's validation and rounding: quantity with material waste to four decimals, net to two decimals, then
gross from the rounded net. Quantity 1, price 0.025, VAT 0.23 → net 0.03, gross 0.04. A preview is
informational; a later write validates the current estimate again.

Call `preview_estimate_budget` with the project, estimate and proposed contingency percent to read the
current stored net total, rounded contingency amount and internal budget. Never send or locally rebuild the
estimate total. The contingency is outside work prices and VAT. Neither preview accepts a tenant ID or
creates an estimate item.
