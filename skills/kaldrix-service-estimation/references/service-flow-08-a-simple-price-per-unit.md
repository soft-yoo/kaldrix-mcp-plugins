### A simple price per unit

Most small contractors price work directly: "wall painting, 25 PLN per m²". Create the service without components
(`create_service_recipe` with `"quantityRule": "WALL_NET_AREA", "unit": "M2", "vatRate": 0.23`, `markupPercent: 0`),
then `add_service_labor_activity` with `name: "Malowanie"`, `priceNet: 25`. Add material norms (paint per m²) as
further components when the user wants them. `add_catalog_service_to_estimate` freezes **one** row per scope with `resourceKind = SERVICE`, `quantity`
equal to the service quantity, the current effective unit price, the markup used, and the composition as
informational `components` (they never change the total).

For every create, update, copy, or service-add operation, send a stable `idempotencyKey`. If transport or output validation
fails after a successful write, retry with the same key and exactly the same input. The server replays the earlier
result for 24 hours and rejects reuse of that key with different input. Replays are shared by all server instances
and survive restarts. On `IDEMPOTENCY_IN_PROGRESS`, read the current state before retrying.

`quantityFactor` defaults to `1`. `UP_TO_WHOLE` rounds the calculated amount up to the next whole resource unit.
For example, 40 m² of painting at a verified consumption of `0.12` liters per m² needs 4.8 liters.
Because LITERS differs from the service unit M2, use a MANUAL component and supply the calculated total
through `componentQuantities`. `UP_TO_WHOLE` rounds liters to 5 liters, not to five containers.
To buy packaged goods, use the resource package-content fields and explicitly convert consumption to whole packages
as described below; the backend does not perform that conversion.
A FIXED brush component stays at one piece.

Recipe reads return the current net cost per billing unit split into material, labor, and equipment. Material waste
uses the component override first and the resource default second. Waste is never applied to labor or equipment.

`defaultElementType` and `quantityRule` describe which BIM element and exact metric can supply the service scope.
For an estimate based on a manually supplied scope, prefer `add_catalog_service_to_estimate`: it loads the complete
recipe on the server, calculates every automatic component, and freezes current prices together. Use
`create_estimation_item` only for a genuinely manual row outside a service recipe.
