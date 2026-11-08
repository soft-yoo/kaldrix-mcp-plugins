## Estimate vs Invoices

The primary estimate (`set_primary_project_estimate`) is the plan the project is settled against. To compare it with
reality, call `get_project_settlement` (scope `settlements.read`, project visible to the person); its `summary` holds the totals. The backend returns, in the saved project currency (PLN when absent):
the primary estimate net/gross and planned direct cost, sales invoiced to the customer with paid, outstanding and
overdue amounts, purchase costs, net balance, margin, share of the estimate already invoiced and share of the planned
cost already spent. Invoices in other currencies are only counted, never converted. Do not recompute these figures
from estimate rows or invoice lists.

Totals include KSeF invoices and manually recorded invoices assigned to the project. When they look incomplete, call `list_invoices_to_assign`,
show its suggestions and assign the invoices the user accepts with `assign_invoice`. A project without a customer can
get one with `set_project_customer`.

The planned cost is the primary estimate's direct cost (material + labor + equipment from service compositions and own
rows). A service priced by hand without a composition adds to the planned income but not to the planned cost, so the
planned margin looks higher than it is; say so when such rows exist.

For the full "Rozliczenie" (settlement) tab of the application call `get_project_settlement` (same scope and access).
It returns the totals in `summary`, plus the project customer, `plan` (primary estimate plan versus invoiced
actuals: planned and actual income, cost and margin, remaining amount to invoice, remaining cost budget, `overBudget`),
suppliers (the `vendors` field) grouped by seller tax ID, `monthly` cash flow for the last 12 months and prioritized `alerts`. Each alert has
a `code`, `severity`, optional `amount`/`percent`/`count` and a `meaning` explaining them. `PLAN_COST_INCOMPLETE` means
some primary-estimate positions are priced without a composition (`count` positions worth `amount` net), so the
planned cost is understated and the planned margin overstated — tell the user and offer to add compositions.
Use `get_project_settlement` when the user asks how the project is doing overall, who owes what, which suppliers cost most or what to do next. Use `list_invoices` with
`projectId` (and `direction`, `overdueOnly`) to show the underlying invoices. Never rebuild the chart or alerts yourself.
