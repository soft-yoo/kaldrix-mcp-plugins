## Totals

- item net total = `quantity * unitPriceNet`;
- item gross total = `totalNet * (1 + vatRate)`; with `vatReducedShare` (proportional 8/23 split) the share of the
  net is taxed at `vatRate` and the rest at 0.23 — `totalGross`, `vatTotals` and `vatSplit` already include it;
- service `compositionCostNet` = only `PER_SERVICE_UNIT` components, for one billing unit. `FIXED` and `MANUAL`
  components are one-off: they cost the same for 5 and for 500 m², so they never enter the price of one unit.
  Price-list `unitPriceNet` = `compositionCostNet` × (1 + markup / 100);
- in the estimate the one-off cost is spread over the quantity of the whole service occurrence, so the frozen
  `unitPriceNet` = (`compositionCostNet` + one-off cost / total service quantity) × (1 + markup / 100). A service
  without one-off components freezes exactly the price-list price;
- estimate M/R/S totals are costs: frozen `components` of service rows by `kind` plus own items of that kind;
- estimate totals are sums of all active rows (service price × quantity), so M + R + S is a cost breakdown, not a
  split of the total.

There are two distinct markup levels. A service recipe can use `markupPercent` to turn composition cost into its catalog
unit price. A project estimate can apply ordered purchase-cost, indirect-cost, and profit rules to rows that inherit its
settings. `contingencyPercent` is separate: it increases the internal budget but never work prices or VAT. There is no
single final-price markup or final-price rounding rule.
