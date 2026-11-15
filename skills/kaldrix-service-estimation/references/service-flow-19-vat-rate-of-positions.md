## VAT on estimate positions

A catalog service supplies the sales VAT of a new service position; a resource sold separately supplies its
own catalog rate. VAT on a material inside a service is purchase information: the customer pays VAT once on
the service's selling price. For a manual position, ask the contractor for its sales rate.

Existing positions keep their saved rate; the contractor may override a draft position. The estimate and the
public offer total position VAT by rate. A private recipient sees the amounts the contractor quoted.

For a Quick quote, omit `vat` to use catalog rates. If the contractor chooses one rate for the quote
positions, send `vat: "CUSTOM"` with a fraction such as `customVatRate: 0.0725` for 7.25%. Taxable zero and
`EXEMPT` are separate choices. To change all positions in a draft, use `set_quick_quote_vat` after the
contractor approves the new rate.
