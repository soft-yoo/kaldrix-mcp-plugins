### Units and packages

Unit arguments accept the `CostUnit` name in any case and common Polish abbreviations (`m²`, `mb`, `m³`, `szt`,
`kpl`, `opak`, `l`, `t`, `rbh`, `h`); results always use the canonical `CostUnit` name. The server never guesses a conversion. Every quantity you send is in the unit of the thing it belongs to: a scope's
`serviceQuantity` in the service unit, a `componentQuantities` entry in the component (resource) unit, and
`sourceQuantity` in its `sourceUnit`. State explicit conversion inputs to the user:

- for model and measurement sources, use `plan_estimate_position_quantity` and copy its ready scopes unchanged.
  Supply explicit thickness when required or a corrected billing total; the server computes the multiplier and
  distributes the total. To change the measured metric, pick the appropriate quantity rule;
- a material sold in bags, buckets or boxes is a resource in `PACK` (or `PCS`) priced per package. Put it in the
  service as `MANUAL` and send the number of packages, or, when the service unit is the same as the resource unit,
  use `PER_SERVICE_UNIT` with `roundingRule: "UP_TO_WHOLE"` to buy whole units only. When the resource has
  `packageContent` + `packageContentUnit` (e.g. 1 bag = 25 `KG`), convert the product's consumption yourself and
  send whole packages: 4 kg/m² × 9 m² = 36 kg → `ceil(36 / 25) = 2` bags in `componentQuantities`. Say the
  conversion to the user; the server never uses package content in calculations;
- material waste (`wastePercent` / `wastePercentOverride`) is added by the server; do not add it again.
