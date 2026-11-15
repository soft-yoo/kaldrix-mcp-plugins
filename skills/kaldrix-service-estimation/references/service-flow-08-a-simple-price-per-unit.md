### A simple price per unit

Small contractors often price work directly: "wall painting, 25 PLN per m²". Create the service without
components (`create_service_recipe` with `"quantityRule": "WALL_NET_AREA", "unit": "M2", "vatRate": 0.23`,
`markupPercent: 0`), then `add_service_labor_activity` with `name: "Malowanie"`, `priceNet: 25`. Add material
norms (paint per m²) as further components when the user wants them. `add_catalog_service_to_estimate`
freezes **one** row per scope with `resourceKind = SERVICE`, `quantity` equal to the service quantity, the
current effective unit price, the markup used and the composition as informational `components` (they
never change the total).

Send a stable `idempotencyKey` with every create, update, copy or service-add operation. If transport or
output validation fails after a successful write, retry with the same key and exactly the same input: the
server replays the earlier result for 24 hours (shared by all server instances, surviving restarts) and
rejects the key with different input. On `IDEMPOTENCY_IN_PROGRESS`, read the current state before retrying.

`quantityFactor` defaults to `1`. `UP_TO_WHOLE` rounds the calculated amount up to the next whole resource
unit. Example: 40 m² of painting at a verified `0.12` liters per m² needs 4.8 liters; LITERS differs from
the service unit M2, so use a MANUAL component with the total in `componentQuantities`. `UP_TO_WHOLE` rounds
liters to 5 liters, not to five containers. For packaged goods use the resource package-content fields and
convert consumption to whole packages yourself ("Units and packages"); the backend does not. A FIXED brush
component stays at one piece.

Recipe reads return the current net cost per billing unit split into material, labor and equipment.
Material waste uses the component override first and the resource default second; it never applies to
labor or equipment.

`defaultElementType` and `quantityRule` define which BIM element and exact metric can supply the scope. For
a manually supplied scope prefer `add_catalog_service_to_estimate`: it loads the complete recipe on the
server, calculates every automatic component and freezes current prices together. Use
`create_estimation_item` only for a genuinely manual row outside a service recipe.
