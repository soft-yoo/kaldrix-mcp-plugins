## The estimate follows the model

After every saved model change (app or MCP) the backend checks each active project estimate in the
background, shortly after the write (not within the same call):

- **Editable draft** (DRAFT, no open offer, no client answer on the current offer version): the backend itself
  corrects quantities of positions measured from the model or a saved measurement when the model changed
  (`QUANTITY_CHANGED`, `changeReason=MODEL_CHANGED`). Prices, services, markups and discounts stay; each
  correction is a "was → is" entry. Never apply these again with `apply_estimate_model_changes` or
  `replace_estimate_position`.
- **Approved, waiting for approval or offered**: nothing changes; only the attention summary updates and the
  app marks the estimate as needing attention. Follow the offer workflow before changing it.
- **Never automatic, in any estimate**: element gone or quantity zero (`SOURCE_REMOVED`, `QUANTITY_ZERO`),
  Kaldrix calculation-method changes (`QUANTITY_CHANGED`, `changeReason=CALCULATION_CHANGED` with explanatory
  `notice`), new matching elements (`newElements`) and quantities not from the model or a measurement. They
  wait for the user's decision.
- **Restoring a project version** corrects drafts the same way within the `restore_project_version` call:
  every editable draft (restored with the version or not, also one whose restore failed) gets the restored
  model's quantities, recorded with the cause "After a version was restored" and the restoring person as
  author. Manual quantities, prices, markups and discounts stay; locked estimates only get the attention
  summary; removed and new elements still wait. If a draft correction fails, the restore still succeeds and the
  draft shows waiting quantities in `review_estimate_against_model`.

After editing the model:

1. Expect drafts to update themselves: zero quantity changes for a draft in `review_estimate_against_model` is
   normal. Right after the write it may still show `QUANTITY_CHANGED` rows the background has not reached;
   read it again before deciding.
2. `get_estimate_model_changes` → tell the user what changed: positions with quantity before/after, the cause
   ("Model change" or "After a version was restored") and who changed or restored the model. Net values
   before/after are null without `price-list.read` consent or for a person outside the company team.
3. Settle the rest with the user, using `itemId` values and `modelVersion` from the latest review:
   - element gone, keep current quantity as manual: `detach_estimate_items_from_model` (`estimates.write`);
   - element gone, remove: `remove_estimate_items_with_missing_source` (`estimates.delete`, `confirm=true` after approval);
   - new elements: add with `apply_estimate_model_changes` (`addNewElementsTo`), or, when one deliberately
     belongs elsewhere (e.g. a socket priced in another position), `skip_estimate_new_elements` (`positionId` =
     `serviceOccurrenceId`, `elementIds` from that position's `newElements`): amounts unchanged, the review stops
     listing it and counts it in `skippedNewElementCount`; adding it later with `replace_estimate_position`
     clears the skip;
   - remaining draft `QUANTITY_CHANGED` rows (e.g. `CALCULATION_CHANGED`): explain whether the drawing
     (`MODEL_CHANGED`) or the calculation method (`CALCULATION_CHANGED`) changed, then after approval
     `apply_estimate_model_changes` with `itemIds`;
   - other sources: `assign_estimate_position_scope` (replace).
   Every `apply_estimate_model_changes` call sends the review's `modelVersion` as `expectedModelVersion` and
   `quantityEngineVersion` as `expectedQuantityEngineVersion`.
4. Once the user has seen the entries, `acknowledge_estimate_model_changes` with their `entryIds` removes them
   from the attention summary (no amount or quantity changes).

An older review's row id may be rejected (`ESTIMATE_MODEL_SOURCE_PRESENT` or a stale `itemId`) because the
background already handled it: re-read the review and use current ids. Detaching and removing work only in an
editable estimate.

## Before sending, approving or generating a PDF

Read `review_estimate_against_model` immediately before `send_estimate_offer`, `update_project_estimate` with
status APPROVED, or `generate_estimate_pdf`. Show its actual findings and unacknowledged automatic changes from
`get_estimate_model_changes`. Decide from `blockingPublicationItemIds` and `hasPublicationWarnings`, never from
counters or cached attention badges.

- Non-empty `blockingPublicationItemIds` = removed or zero-quantity model sources: correct, remove or detach
  them first. `ESTIMATE_MODEL_SOURCES_INVALID` cannot be overridden.
- Other model warnings or unacknowledged automatic quantity changes need the user's explicit acceptance; only
  then send `confirmModelWarnings=true`, that review's `modelVersion` as `expectedModelVersion` and
  `quantityEngineVersion` as `expectedQuantityEngineVersion` on the send, approval or PDF call, plus its ordinary
  `confirm=true`. Without warning consent: `ESTIMATE_MODEL_WARNINGS_UNCONFIRMED`.
- Missing version with warning consent, changed model or changed calculation method (`quantityEngineVersion`):
  `ESTIMATE_MODEL_REVIEW_STALE`. Read and explain the new review, ask for a new decision. Never swap in a newer
  version number in a previously approved request, retry silently, or treat `confirm=true` as warning consent.

The backend rechecks the model on each operation. Confirmation accepts the reviewed warnings for that operation
only; it neither removes findings nor acknowledges change-history entries.
