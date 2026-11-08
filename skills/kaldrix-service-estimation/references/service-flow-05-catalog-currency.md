## Catalog Currency

These rules apply to both personal accounts and companies. A private user does not need to create a company.
Both personal and company account owners can change the default currency in **Settings** (`/settings`).
Company owners can also use **Company settings → Project and price list currency** (`/tenant/settings#currency`). Only the active account owner
can change this default (an owner privilege, not a scope); other company members can read and use it. Both save the same account-scoped default used by the backend.

- Start with `list_price_lists`: `defaultCurrency` is the current account setting for NEW lists and projects;
  `currencyOptions` lists supported currencies and `requiresEurExchangeRate`. Currently these are PLN, EUR and DKK.
  Omit `currency` in `create_price_list` to use the company setting automatically; an unset setting means PLN.
  Do not ask the user to repeat a configured currency. An explicit override is only for a requested different currency.
- Each existing price list has its own stored `currency` and optional `eurExchangeRate` in `get_price_list`.
  This is authoritative for its resource, labor and service prices, even when `defaultCurrency` now differs.
  Catalog writes without `priceListId` use the DEFAULT PRICE LIST and its stored currency, not the current company setting.
  Resource reads and writes return `priceListId` and `currency` alongside `priceNet`; keep them together when displaying prices.
- `create_catalog_resource`, `update_catalog_resource` and `add_service_labor_activity` accept prices already
  expressed in the target price-list currency. Example: `priceNet=100` in DKK means 100 DKK.
- Creating an empty price list, adding manual prices and creating an estimate never require an exchange rate.
  `requiresEurExchangeRate` describes EUR starter-price import, not ordinary catalog creation.
- PLN and EUR starter prices are native catalog prices. Other currencies use EUR starter prices:
  call `install_construction_starter_pack` with `exchangeRate=7.46` for **1 EUR = 7.46 DKK**.
  A saved EUR rate can be reused, but never invent a missing rate. Existing resources keep their prices.
- `copy_price_list` converts a whole catalog in the backend. Omitted `currency` preserves source currency.
  When changing currency, provide `exchangeRate`: target units for one source unit, e.g. 0.57 for DKK to PLN.
  No EUR metadata is needed with this direct rate. The source stays unchanged.
- `import_price_list` accepts source prices plus their `currency` and an explicit `exchangeRate` when target
  currency differs. The backend converts resource prices once; service compositions use those resources.
  Quantities, VAT and markup are unchanged. Same-currency imports require no rate.
- Estimates inherit their PROJECT currency. Select a matching price list; changing an estimate's price list,
  comparing lists requires the same currency. Copying a single service into another currency requires its own
  explicit exchangeRate; existing target resources keep their prices, while missing resources are copied and converted. If no matching catalog exists,
  the backend creates an empty matching catalog without inventing any exchange rate.
- Replacement resources, manual rows linked to catalog resources and sheet components must belong to the estimate's
  price list (`PRICE_LIST_RESOURCE_OTHER_PRICE_LIST` otherwise). Matching units and kinds alone are insufficient.
  Convert a complete catalog first, then use its resources; never relabel a resource price from another currency.
