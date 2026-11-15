## Contacts and invoice roles

The company UI groups these records under **Contacts** (`CUSTOMERS` module), always relative to the current company;
one company can have both roles:

- **Customers**: people and companies the current company invoices for sales (`SALES`); they can have projects and offers before the first invoice.
- **Suppliers**: companies issuing cost invoices to the current company (`PURCHASE`), from KSeF or manually recorded documents.

An invoice supplier is not a wholesaler workspace of type `SUPPLIER` or its material catalog. Names stay as the runtime
schema exposes them: `list_customers`, `create_customer`, `customers.*`, `customerId`, `vendorId`; a `vendorId` or
`vendors` field refers to an invoice supplier. Purchase invoice suppliers derive from the seller tax ID; there is no
supplier-address-book write tool and no public wholesaler catalog in MCP.
