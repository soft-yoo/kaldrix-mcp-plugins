## Exported and shared amounts

- PDF, Excel, public offer links and Kaldrix Rynek preserve the currency saved on the estimate or offer snapshot.
  Changing account settings never converts or relabels them. Conversion happens explicitly during catalog copying, file import or starter-price import.
- Service recipe responses include `priceListId` and `currency`, including pages spanning several lists. Estimate item
  pages include the saved estimate `currency` for every row and total. Keep these values with monetary amounts.
- Price-list Excel files record `currency` and `eurExchangeRate` in the `Kaldrix` sheet. Send the source currency
  to `import_price_list`; different target currency requires `exchangeRate`. A missing or invalid rate is rejected
  before preview or writes (`PRICE_LIST_CONVERSION_RATE_REQUIRED`). Never change the source currency label to hide a mismatch.
- Kaldrix Rynek records accepted quote amounts with the quote currency, including EUR and DKK; it does not convert them to PLN.
