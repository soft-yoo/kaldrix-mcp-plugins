### Manual vs model quantities

A model scope stays linked to its element or measurement: the estimate remembers the model version, an
editable draft corrects the quantity by itself after a model change, and `review_estimate_against_model`
reports the rest (`QUANTITY_CHANGED` in an approved or offered estimate, `SOURCE_REMOVED`). A manual quantity
(`serviceQuantity` alone) is the user's number and is never compared with the model. Prefer model scopes for
everything the model measures (walls, rooms, slabs, openings, installations, saved measurements) and manual
quantities for work outside the model. Never type a number from a navigation tool as a manual quantity when a
model scope exists — it would silently stop following the model.

**Quantity formula (wyliczenie ilości).** `list_estimate_quantity_sources` returns `quantityFormula` for a
source and estimate rows carry it for model scopes, e.g. `4,20 × 2,70 − 0,90 × 2,05 = 9,49 m²`,
`Krokiew: 22 szt. × 3,93 m = 86,46 mb`, `trasa 62,10 m + 10% zapasu = 68,31 mb`. Quote it when explaining a
quantity. It is absent for manual quantities, when it adds nothing (1 szt.) or when the model quantity changed
after the scope was frozen (then `review_estimate_against_model` reports the change).

**Pricing basis (Podstawa).** A service may carry `basis` — a KNR / KNNR code (`KNR 2-02 0126-01`) or
`kalkulacja własna` — set with `create_service_recipe` / `update_service_recipe` / `import_price_list`. A new
position copies it as `knrCode`, printed in the detailed PDF. It is the company's own text; never invent KNR codes.
