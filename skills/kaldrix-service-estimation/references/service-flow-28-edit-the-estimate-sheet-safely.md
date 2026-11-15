
## Edit the estimate sheet safely

`list_estimate_sheet_rows` gives exact scope versions, planned and completed work, KNR codes, document order,
extra text cells, markup inheritance and backend price breakdowns. It needs `estimates.read` and a person who
can see company amounts (`price-list.read`); others get FORBIDDEN. `list_estimation_items` remains a compact
read contract.

Collect all scope rows of one `serviceOccurrenceId`, across pages if needed (an own item has one scope). Send
one patch per existing position to `update_estimate_sheet` with its complete exact `versions` list. The writer
requires `estimates.write` (the token's and the person's), `confirm=true` on every call and a DRAFT estimate;
an APPROVED estimate accepts only `actualQuantity` or `clearActualQuantity`. One atomic batch holds 1 to 200
position patches; a stale or incomplete version set, a duplicate position or an invalid later row rejects the
whole batch.

The backend distributes occurrence quantities over scopes and computes selling prices, taxes, completed
totals and markups. Planned quantity changes in the sheet are allowed only for one manual scope; to change a
linked quantity or the planned quantity of a multi-scope position, edit its scopes. Components in the read
result are per-scope quantities; a replacement `components` list describes the whole service occurrence, in
the component's unit (no implicit conversion). Omit a field to keep it; use explicit clear flags for
completed quantity, manual order or section. Service rows use `CUSTOM` (manual net selling price) or
`COMPOSITION` (backend composition calculator). Keep user-entered names, KNR codes and cell texts in their
original language. RESOURCES estimates: change an own LABOR, MATERIAL or EQUIPMENT row's net purchase price
with `purchasePriceNet` (a price change through `unitPriceNet` is rejected; see
[Project Estimate Flow](service-flow-13-project-estimate-flow-full-catalog-tools.md)).

This tool edits existing positions only; it does not copy or deactivate positions. After a conflict or an
uncertain network result, read the current rows before a new edit. Never replace the versions of an old
proposal automatically, and never retry creating a copy through another tool without checking its
idempotency contract.

### Authoritative financial summaries

`list_estimation_items` returns `costTotalNet`, `markupTotalNet`, `uncostedNet`, `markupOnCostPercent`,
`vatTotal` and `vatTotals` (rate, net, VAT, gross), covering all matching active rows before pagination.
Hand-priced services without a composition are excluded from markup; the percentage is null when direct cost
is zero or hidden. Reuse these values instead of summing one response page. VAT buckets use the same server
calculation as PDF and web Excel exports. The read does not modify frozen prices.
