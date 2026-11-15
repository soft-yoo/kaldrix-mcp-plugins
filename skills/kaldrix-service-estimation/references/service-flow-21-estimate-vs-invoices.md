## Estimate vs Invoices

The primary estimate (`set_primary_project_estimate`) is the plan the project is settled against.
`get_project_settlement` (scope `settlements.read`, project visible to the person) is the application's
"Rozliczenie" (settlement) tab, in the saved project currency (PLN when absent):

- `summary`: primary estimate net/gross and planned direct cost, sales invoiced to the customer with paid,
  outstanding and overdue amounts, purchase costs, net balance, margin, share of the estimate already invoiced
  and share of the planned cost already spent. Invoices in other currencies are only counted, never converted;
- the project customer and `plan` (primary estimate plan versus invoiced actuals: planned and actual income,
  cost and margin, remaining amount to invoice, remaining cost budget, `overBudget`);
- suppliers (`vendors`) grouped by seller tax ID, `monthly` cash flow for the last 12 months and prioritized
  `alerts`, each with `code`, `severity`, optional `amount`/`percent`/`count` and an explanatory `meaning`.

Use it when the user asks how the project is doing, who owes what, which suppliers cost most or what to do
next. Never recompute these figures from estimate rows or invoice lists, or rebuild the chart or alerts. Use
`list_invoices` with `projectId` (and `direction`, `overdueOnly`) for the underlying invoices.

Totals include KSeF invoices and manually recorded invoices assigned to the project. When they look
incomplete, call `list_invoices_to_assign`, show its suggestions and assign the accepted ones with
`assign_invoice`. A project without a customer can get one with `set_project_customer`.

The planned cost is the primary estimate's direct cost (material + labor + equipment from service
compositions and own rows). A service priced by hand without a composition adds to planned income but not to
planned cost, so the planned margin looks higher than it is; say so when such rows exist.
`PLAN_COST_INCOMPLETE` means `count` primary-estimate positions worth `amount` net are priced without a
composition: planned cost is understated and planned margin overstated — tell the user and offer to add
compositions.
