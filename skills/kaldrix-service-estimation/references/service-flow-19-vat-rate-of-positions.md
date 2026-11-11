## VAT on estimate positions

A catalog service supplies the sales VAT for a new service position. A resource sold separately supplies its own catalog rate. VAT on a material inside a service is purchase information. The customer pays VAT once on the selling price of the service. For a manual position, ask the contractor for its sales rate.

Use the saved rate for existing positions. The contractor may override a draft position. The estimate and the public offer total the VAT of the positions by rate. A private recipient sees the amounts the contractor quoted.

For a Quick quote, omit `vat` to use the catalog rates. If the contractor chooses one rate for the quote positions, send `vat: "CUSTOM"` with a fraction such as `customVatRate: 0.0725` for 7.25%. Taxable zero and `EXEMPT` are separate choices. To change all positions in a draft, use `set_quick_quote_vat` after the contractor approves the new rate.
