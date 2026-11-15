## Exported and shared amounts

- PDF, Excel, public offer links and Kaldrix Rynek keep the currency saved on the estimate or offer snapshot;
  account settings never convert or relabel them. Conversion happens only explicitly: catalog copy, file
  import or starter-price import.
- Service recipe responses include `priceListId` and `currency`, also on pages spanning several lists;
  estimate item pages include the saved estimate `currency` for every row and total. Keep them with amounts.
- Price-list Excel files record `currency` and `eurExchangeRate` in the `Kaldrix` sheet. Send the source
  currency to `import_price_list`; a different target currency requires `exchangeRate`. A missing or invalid
  rate is rejected before preview or writes (`PRICE_LIST_CONVERSION_RATE_REQUIRED`). Never change the source
  currency label to hide a mismatch.
- Kaldrix Rynek records accepted quote amounts in the quote currency, including EUR and DKK, without
  converting them to PLN.
