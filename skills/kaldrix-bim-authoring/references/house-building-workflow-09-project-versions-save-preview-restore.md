## Project versions: save, preview and restore

A project version is a saved copy of the whole model and of the project's draft estimates. Versions exist in every
plan; the person sees them in the app under **Wersje / Versions**. Single saved changes have no undo: going back
always means restoring a version.

**Automatic versions.** The backend saves one by itself before a large change (removing a floor, a quick house,
replacing or removing a roof, applying a roof framing draft, a batch or selection that removes 10 or more elements),
every 20 model versions and before every restore. Writes made through MCP create them too, marked as made by an AI
agent. Do not rely on an automatic version for your own risky plan: it exists only for those operations.

**Save before a large or risky change.** Before many removals, rebuilding a floor or outline, replacing a roof or a
large `apply_geometry_batch`, you may call `save_project_version` with a short name the person will recognise, e.g.
"Before the roof" (1-80 characters). It needs `history.write` and `projects.read`. Tell the user you saved it. With
no change since the newest version, that version gets the name instead of a new copy. At the plan's limit of manual
versions the call fails with `RESTORE_POINT_MANUAL_LIMIT_REACHED`: tell the user; deleting a manual version is done
in the app. `list_project_versions` shows the versions and the plan's counters and limits; it and
`preview_project_version` need `history.read` and `projects.read`.

**Go back after a mistake.**

1. `list_project_versions` → choose the version with the user (reason, name, author, time, model version). Never
   choose by guessing when several fit.
2. `preview_project_version` → show the user in plain words what changes in the model per element kind, what happens
   to each estimate (which drafts come back, which do not and why) and any blockers. The preview changes nothing.
3. Only after the user's explicit yes: `restore_project_version` with `confirm=true`, a new `operationId` (UUID, one
   per restore), `expectedModelVersion`, `expectedSceneVersion` and, with `restoreEstimates=true`,
   `expectedEstimateStamps` copied unchanged from the preview. It needs `history.restore` and `projects.write`;
   restoring estimates also needs `estimates.write`. The consent screen lists `history.restore` with removals,
   unchecked by default. Called without `confirm=true` the tool only returns the current preview.
4. Read back `get_project_detail` (the restore is a new model version, not a rewind) and the estimates
   (`get_estimate_model_changes` shows the quantities corrected after the restore).

Refusals: `RESTORE_POINT_PROJECT_CHANGED` / `RESTORE_POINT_ESTIMATE_CHANGED` — the project or an estimate changed
since the preview: preview again and ask again if the effect differs. `RESTORE_POINT_NOT_RESTORABLE` — explain the
blockers from the preview. `RESTORE_POINT_NOT_FOUND` — the version belongs to another project. After an uncertain
response or `RESTORE_POINT_INCOMPLETE`, retry with the same `operationId` and the same input: the model is never
restored twice and the unfinished part completes. If the model changed in between, estimates not restored yet stay
current (`restored=false` with the reason) and already follow the newer model.

The state before the restore is saved as a new version, so a wrong restore can itself be undone the same way.
Estimates approved, waiting for approval or offered never change. After every restore, every editable draft —
restored with the version or not — gets the model-counted quantities of the restored model within the same call,
recorded in `get_estimate_model_changes` as "After a version was restored"; do not apply them again. Manual
quantities, prices and markups stay. An estimate with `restored=false` keeps its current items and prices (only the
model-counted quantities follow) — compare it with the project together with the user. Review what is left (removed
or new elements, locked estimates, and any draft whose quantities still wait) with `review_estimate_against_model`
and apply changes only after the user approves.
