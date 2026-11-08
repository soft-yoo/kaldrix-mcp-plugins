## Estimate PDF

`generate_estimate_pdf` (scope `estimates.write`, the same scope the application requires) renders
the estimate on the server and stores the PDF with the project documents, category `REPORTS`. It takes `projectId`,
`estimateId`, optional `variant` and `confirm=true` after the user approves, and returns `documentId`, `fileName`, `sizeBytes`, a temporary `downloadUrl` with `expiresAt`, and
`openInApp` — where the user finds the file in the application (project → Dokumenty → Raporty). Give the user that
place; when the link expires, call `get_document_download_access` with `documentId`.

- `variant` STANDARD (default) follows the estimate's saved view (`presentationMode`), exactly like the application:
  `SIMPLE` prints the client offer (no internal costs), `DETAILED` prints the detailed estimate with KNR, scopes,
  composition and unit-price calculation. A DETAILED PDF is an internal document (`internalOnly = true`): never present
  it as the client offer. `CLIENT` always prints the client offer with sale prices only, `QUANTITIES` a bill of
  quantities without prices. Without access to company costs, STANDARD on a DETAILED estimate prints the client offer.
  The saved view is set with `update_project_estimate` (`presentationMode`).
- An `APPROVED` estimate returns its stored PDF. A `DRAFT` or `PENDING_APPROVAL` estimate gets a fresh file on every call,
  which replaces its previous PDF, so generate it once the estimate is ready.
- An estimate without rows cannot be printed. The PDF prints the take-off (przedmiar) quantities; amounts, VAT per rate
  and section subtotals come from the server, in the saved estimate currency. Printed amounts and quantities are rounded
  half-up (money to 2 places, quantities up to 4), like the estimate itself. In the detailed PDF a scope shows its
  source name (or "from the model") only when it points at a model element or a saved measurement; every other
  quantity is labelled as entered by hand. Under a model scope the detailed PDF prints its quantity formula
  (przedmiar), and a detailed estimate with its own markups prints an "Estimate assumptions" block (Kz / Kp / Z with
  their basis).
- There is no Excel tool. The application's "Pobierz Excel" builds the workbook in the browser with the same rules
  as the PDF (source labels, VAT per rate, section totals).
