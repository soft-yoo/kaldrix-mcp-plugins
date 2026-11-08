## Contacts and invoice roles

The company UI groups these records under **Contacts** (`CUSTOMERS` module):

- **Customers** are the people and companies the current company invoices for sales (`SALES`). They can also have projects and offers before the first invoice.
- **Suppliers** are the companies that issue cost invoices to the current company (`PURCHASE`), from KSeF or manually recorded documents.

The same company can have both roles. Roles are always relative to the current company.
The invoice supplier is distinct from a wholesaler workspace of type `SUPPLIER` and its material catalog.
Tool names, scopes and field names remain stable: use `list_customers`, `create_customer`, `customers.*`,
`customerId` and `vendorId` exactly as exposed by the runtime schema. A `vendorId` or `vendors` field
refers to an invoice supplier. Purchase invoice suppliers are derived from the seller tax ID;
there is no separate supplier-address-book write tool.
