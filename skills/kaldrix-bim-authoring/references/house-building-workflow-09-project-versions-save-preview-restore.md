## Project versions: save, preview and restore

A project version is a saved copy of the whole model and the project's draft estimates. Versions exist in every
plan; the person sees them in the app under **Wersje / Versions**. Single saved changes have no undo: going back
always means restoring a version.

**Automatic versions.** The backend saves one before a large change (removing a floor, a quick house, replacing or
removing a roof, applying a roof framing draft, a batch or selection that removes 10 or more elements), every 20
model versions and before every restore, also for MCP writes (marked as made by an AI agent). Do not rely on them
for your own risky plan.

**Save before a large or risky change** (many removals, rebuilding a floor or outline, replacing a roof, a large
`apply_geometry_batch`): you may call `save_project_version` with a short recognisable name, e.g. "Before the roof"
(1-80 characters); needs `history.write` and `projects.read`. Tell the user. With no change since the newest
version, that version gets the name instead of a new copy. At the plan's manual-version limit it fails with
`RESTORE_POINT_MANUAL_LIMIT_REACHED`: tell the user; manual versions are deleted in the app.
`list_project_versions` (versions, plan counters and limits) and `preview_project_version` need `history.read`
and `projects.read`.

**Go back after a mistake.**

1. `list_project_versions` → choose the version with the user (reason, name, author, time, model version); never
   guess when several fit.
2. `preview_project_version` → explain in plain words the model changes per element kind, what happens to each
   estimate (which drafts come back, which do not and why) and any blockers. The preview changes nothing.
3. Only after the user's explicit yes: `restore_project_version` with `confirm=true`, a new `operationId` (UUID, one
   per restore), `expectedModelVersion`, `expectedSceneVersion` and, with `restoreEstimates=true`,
   `expectedEstimateStamps` copied unchanged from the preview. Needs `history.restore` and `projects.write`;
   restoring estimates also `estimates.write`. The consent screen lists `history.restore` with removals, unchecked
   by default. Without `confirm=true` the tool only returns the current preview.
4. Read back `get_project_detail` (the restore is a new model version, not a rewind) and the estimates
   (`get_estimate_model_changes` shows quantities corrected after the restore).

Refusals: `RESTORE_POINT_PROJECT_CHANGED` / `RESTORE_POINT_ESTIMATE_CHANGED` — project or estimate changed since the
preview: preview again and ask again if the effect differs. `RESTORE_POINT_NOT_RESTORABLE` — explain the preview's
blockers. `RESTORE_POINT_NOT_FOUND` — the version belongs to another project. After an uncertain response or
`RESTORE_POINT_INCOMPLETE`, retry with the same `operationId` and input: the model is never restored twice and the
unfinished part completes. If the model changed in between, estimates not restored yet stay current
(`restored=false` with the reason) and already follow the newer model.

The pre-restore state is saved as a new version, so a wrong restore can be undone the same way. Approved, awaiting
approval or offered estimates never change. After every restore, every editable draft (restored or not) gets the
model-counted quantities of the restored model in the same call, recorded in `get_estimate_model_changes` as
"After a version was restored"; do not apply them again. Manual quantities, prices and markups stay. An estimate
with `restored=false` keeps its items and prices (only model-counted quantities follow); compare it with the
project together with the user. Review what is left (removed or new elements, locked estimates, drafts whose
quantities still wait) with `review_estimate_against_model` and apply changes only after the user approves.
