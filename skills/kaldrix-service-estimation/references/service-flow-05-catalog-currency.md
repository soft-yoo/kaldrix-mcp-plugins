## Catalog Currency

Applies to personal accounts and companies; a private user needs no company. Personal and company account
owners change the default currency in **Settings** (`/settings`); company owners also in **Company settings →
Project and price list currency** (`/tenant/settings#currency`). Only the active account owner can change it
(an owner privilege, not a scope); other company members read and use it. Both save the same account-scoped
default used by the backend.

- `list_price_lists`: `defaultCurrency` is the current account setting for NEW lists and projects;
  `currencyOptions` lists supported currencies (currently PLN, EUR and DKK) and `requiresEurExchangeRate`.
  Omit `currency` in `create_price_list` to use the company setting (unset = PLN). Do not ask the user to
  repeat a configured currency; override only for a requested different currency.
- Each existing price list stores its own `currency` and optional `eurExchangeRate` (`get_price_list`),
  authoritative for its resource, labor and service prices even when `defaultCurrency` now differs. Catalog
  writes without `priceListId` use the DEFAULT PRICE LIST and its stored currency, not the company setting.
  Resource reads and writes return `priceListId` and `currency` with `priceNet`; display them together.
- `create_catalog_resource`, `update_catalog_resource` and `add_service_labor_activity` take prices already in
  the target price-list currency: `priceNet=100` in DKK means 100 DKK.
- An empty price list, manual prices and a new estimate never need an exchange rate.
  `requiresEurExchangeRate` concerns EUR starter-price import, not ordinary catalog creation.
- PLN and EUR starter prices are native catalog prices; other currencies use EUR starter prices:
  `install_construction_starter_pack` with `exchangeRate=7.46` for **1 EUR = 7.46 DKK**. A saved EUR rate can
  be reused; never invent a missing rate. Existing resources keep their prices.
- `copy_price_list` converts a whole catalog in the backend; omitted `currency` keeps the source currency.
  When changing currency send `exchangeRate` = target units per one source unit (e.g. 0.57 for DKK to PLN);
  no EUR metadata is needed with this direct rate. The source stays unchanged.
- `import_price_list` takes source prices with their `currency` and an explicit `exchangeRate` when the
  target currency differs. The backend converts resource prices once; service compositions use those
  resources. Quantities, VAT and markup are unchanged; same-currency imports need no rate.
- Estimates inherit their PROJECT currency. Select a matching price list; changing an estimate's price list
  or comparing lists requires the same currency. Copying one service into another currency needs its own
  explicit exchangeRate; existing target resources keep their prices, missing ones are copied and converted.
  Without a matching catalog the backend creates an empty one without inventing an exchange rate.
- Replacement resources, manual rows linked to catalog resources and sheet components must belong to the
  estimate's price list (else `PRICE_LIST_RESOURCE_OTHER_PRICE_LIST`); matching units and kinds are not
  enough. Convert a complete catalog first, then use its resources; never relabel a resource price from
  another currency.
