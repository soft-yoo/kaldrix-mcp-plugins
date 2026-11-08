## The estimate follows the model

After every saved model change (from the app or from MCP) the backend checks each active estimate of the project
in the background, shortly after the write and not within the same call:

- **Editable draft** (DRAFT, no open offer and no client answer on the current offer version): the backend corrects
  by itself the quantities of positions measured from the model or from a saved measurement (`QUANTITY_CHANGED`).
  Prices, services, markups and discounts stay. Each correction is recorded as a "was → is" entry. Do not apply these
  quantities again with `apply_estimate_model_changes` or `replace_estimate_position`.
- **Approved, waiting for approval or offered estimate**: nothing changes. The backend only updates its attention
  summary; the app marks the estimate as needing attention. Follow the offer workflow before changing it.
- **Never automatic, in any estimate**: positions whose element left the model or whose quantity dropped to zero
  (`SOURCE_REMOVED`, `QUANTITY_ZERO`), new matching model elements (`newElements`) and quantities that do not come
  from the model or a measurement. They wait for the user's decision.
- **Restoring a project version** corrects drafts the same way, within the `restore_project_version` call itself:
  every editable draft, restored with the version or not (also one whose restore failed), gets the model-counted
  quantities of the restored model, recorded as an entry with the cause "After a version was restored" and the person
  who restored as the author. Manual quantities, prices, markups and discounts stay; locked estimates only get the
  attention summary; removed and new elements still wait for the user. In the rare case the correction of a draft
  fails, the restore still succeeds and the draft shows quantities waiting in `review_estimate_against_model`.

After editing the model:

1. Expect drafts to update themselves. `review_estimate_against_model` can show zero quantity changes for a draft:
   that is the normal result, not a failure. Right after the write it can still show `QUANTITY_CHANGED` rows the
   background has not reached yet; read it again before deciding.
2. `get_estimate_model_changes` → tell the user what changed: positions with quantity before and after, the cause
   ("Model change" or "After a version was restored") and who changed or restored the model. Net values before and after are null without `price-list.read` consent or for a person
   outside the company team.
3. Handle what is left with the user, using `itemId` values from the latest review:
   - keep a position whose element is gone, with its current quantity as a manual quantity:
     `detach_estimate_items_from_model` (`estimates.write`);
   - remove such positions: `remove_estimate_items_with_missing_source` (`estimates.delete`, `confirm=true` after the
     user approves);
   - new elements: add them with `apply_estimate_model_changes` (`addNewElementsTo`) or skip them with
     `skip_estimate_new_elements`;
   - remaining `QUANTITY_CHANGED` rows in a draft (quantities the backend does not correct itself): `apply_estimate_model_changes`
     with `itemIds`, after approval.
4. When the user has seen the entries, `acknowledge_estimate_model_changes` with their `entryIds` removes them from
   the attention summary. It changes no amount or quantity.

A row id from an older review can be rejected with a validation error (`ESTIMATE_MODEL_SOURCE_PRESENT` or a stale
`itemId`) because the background already handled it: read the review again and use its current ids. Detaching and
removing work only in an editable estimate.
