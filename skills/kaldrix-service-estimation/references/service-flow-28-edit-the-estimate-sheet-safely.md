
## Edit the estimate sheet safely

Use `list_estimate_sheet_rows` for exact scope versions, planned and completed work,
KNR codes, document order, extra text cells, markup inheritance and backend price breakdowns.
It needs `estimates.read` and a person who can see company amounts (`price-list.read`); people without
access to amounts get FORBIDDEN.
The existing `list_estimation_items` remains a compact read contract.

Collect all scope rows of one `serviceOccurrenceId`, across pages if needed. An own item
has one scope. Send one patch per existing position to `update_estimate_sheet` and include
its complete exact `versions` list.
The writer requires `estimates.write` (the token's and the person's), `confirm=true` on every call and a DRAFT
estimate; an APPROVED estimate accepts only `actualQuantity` or `clearActualQuantity`.
There can be 1 to 200 position patches in one atomic batch. A stale or incomplete version
set, duplicate position or invalid later row rejects the whole batch.

The backend distributes occurrence quantities over scopes and computes selling prices,
taxes, completed totals and markups. Planned quantity changes in the sheet are allowed only
for one manual scope. To change a linked quantity or the planned quantity of a multi-scope
position, edit its scopes instead. Components in the read result are per-scope quantities;
a replacement `components` list describes the whole service occurrence. Use the component's
unit; no implicit unit conversion is performed. Omit a field to retain it, and use explicit
clear flags for completed quantity, manual order or section. For price mode use `CUSTOM`
(manual selling price) or `COMPOSITION` (the common backend calculator). Keep user-entered
names, KNR codes and cell texts in their original language.

This tool edits existing positions only. It does not copy or deactivate positions.
After a conflict or an uncertain network result, read the current rows before a new edit.
Do not replace the versions of an old proposal automatically, and do not retry creating a
copy through another tool without checking its idempotency contract.

### Authoritative financial summaries

`list_estimation_items` returns `costTotalNet`, `markupTotalNet`, `uncostedNet`,
`markupOnCostPercent`, `vatTotal`, and `vatTotals` (rate, net, VAT, gross).
These summaries include all matching active rows before pagination. Hand-priced
services without a composition are excluded from markup; the percentage is null
when direct cost is zero or hidden. Reuse these values for reporting rather than
summing one response page. VAT buckets use the same server calculation as PDF and
web Excel exports. The read does not modify frozen prices.
