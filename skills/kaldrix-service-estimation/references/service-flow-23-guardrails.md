## Guardrails

- Never send or invent a tenant ID. Tenant identity comes from the authenticated MCP context.
- Never invent resource, recipe, project, or estimate UUIDs; obtain them from list/create tools.
- Never claim that prices synchronize with suppliers.
- Never invent BIM source values. BIM-linked writes are accepted only when the source and metric match the current model.
- Never choose a quantity rule from the unit alone; identify what project element and metric the service measures.
- Never convert units silently. Quantities are sent in the service or component unit; explain any conversion
  (e.g. m² of tiles → bags of adhesive) to the user before sending it.
- Never change the primary estimate or deactivate an estimate without the user's explicit consent.
- Never apply hidden markups or rounding.
- Never replace estimate markup rules before reading `kaldrix://estimation/markup-rules` and confirming the intended rates and bases.
- Never mix price lists: a service from one price list cannot be added to an estimate on another price list, and its
  components must come from its own price list. Check `priceListId` before writing.
- Never recompute a room area from the drawn outline or from width × length; use the quantities returned by the model.
- Changing `defaultMarkupPercent` reprices every service without its own markup and starts a new `catalogVersion`.
- Treat service components as references to current resources and estimate items as frozen pricing snapshots with
  controlled price/VAT/note overrides.
- A service never has its own price: for a quick price per unit add one labour activity (`add_service_labor_activity`).
