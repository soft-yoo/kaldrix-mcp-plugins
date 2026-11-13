## Manual item and budget previews

Call `preview_manual_estimate_item` before `create_estimation_item` when the user supplies a
custom service or a separate material, labor or equipment cost. Send the same project, estimate,
name, kind, quantity, unit, price, VAT, optional resource, section and material waste to both. In a RESOURCES
estimate send the purchase price as `purchasePriceNet`; outside RESOURCES send the sale price as `unitPriceNet`.
The preview does not write data and uses the write path's validation and rounding: quantity with
material waste is rounded to four decimals, net to two decimals, then gross is computed from
rounded net. For quantity 1, price 0.025 and VAT 0.23, the result is net 0.03 and gross 0.04.
Treat a preview as informational; a later write validates the current estimate again.

Call `preview_estimate_budget` with the project, estimate and proposed contingency percent to
read the current stored net total, rounded contingency amount and internal budget. Do not send
or locally reconstruct the estimate total. The contingency is outside work prices and VAT.
Neither preview accepts a tenant ID or creates an estimate item.
