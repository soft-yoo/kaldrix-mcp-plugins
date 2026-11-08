## Offer workflow metadata

Use `get_estimate_offer_workflow` with `estimateId` to inspect server-owned action availability,
form limits and validity preview. `latestOffer` includes the recipient e-mail and client answer details only with
`customers.read`; the client link (`shareUrl`) is never returned here.
Respect `canSend`, `canResend`, `canRevoke` and `sendUnavailableReason`; enabled modules,
the person's permissions and current document state decide availability. The preview uses the same calendar
rule as PDF generation and actual send. Without `estimates.write`, every write action is disabled
and `sendUnavailableReason` is `WRITE_SCOPE_REQUIRED` even when the person has that permission in the app.
Sending recalculates from its execution time.
This read-only tool does not send an email or share a link.


### Project workflow

Use `get_project_workflow` with `projectId` to read the backend-selected stage and ordered steps.
The same result is exposed as `ProjectDto.workflow` and `projectWorkflow` in GraphQL.
Follow `nextStepId` only when present, and use `actionEnabled` rather than inferring permission
from the person's permissions or document status. Missing modules hide their steps; read failures remain errors.
OAuth scopes additionally limit the result: `projects.read` alone returns project steps only;
`estimates.read`, `customers.read`, `construction-log.read` and `invoices.read` enable their
respective domain facts. Actions require the corresponding write scope, both granted to the connector and held by the person.
This read does not create estimates, send offers or change the model.

### After the offer is accepted

The accepted estimate becomes the primary one and the contract plan: prices and contract quantities are frozen and
`update_estimate_sheet` accepts only executed quantities (`actualQuantity` or `clearActualQuantity`). Executed
quantities drive the progress of the construction stages created from estimate sections (`get_project_schedule`) and
the stage instalments of the contract. The contract itself comes from the accepted offer (`list_project_contracts`,
`create_project_contract`, `get_project_contract` — one contract per accepted offer, several in a project; default instalments 20% at signing, 70% after the estimate-section stages, 10% after the final
handover). The whole process after the offer (start of works, stages, acceptance protocols, completion) is described in
`kaldrix://mcp/agent-workflow`, section "From the accepted offer to the keys". A renovation without a drawing
(`workType=RENOVATION`) is priced with manual quantities and goes through the same process.

### Bulk service action metadata

Use `plan_estimate_element_batch` for one element and the visible active catalog recipe IDs
(1 to 2000 distinct IDs). Supply the current modelVersion. Follow `actionEnabled`, pending
recipe IDs and `maxSelection`; do not recalculate duplicates or quantities in the client.
Services with manual components are returned as `manualRecipeIds` and must be added separately
with explicit manual quantities. Archived price lists and non-draft estimates disable the action.
Without `estimates.write`, the tool returns WRITE_SCOPE_REQUIRED. This tool is read-only.
A plan is not authorization to write; actual operations validate current quantities and permissions.

### Price the model by stages at once

`price_estimate_from_model` runs the stages SHELL, FINISHING and INSTALLATIONS (all by default, or the ones in `stages`)
in that order and reports each stage separately: PRICED, NOTHING_TO_PRICE (no elements of that stage), FAILED with its
`errorCode` (earlier stages stay saved) or NOT_RUN. PRICED means the stage was processed; it can add zero positions and still report unmatched or manual elements. Inspect those lists and `get_estimate_checks` before reporting full coverage. The English MCP presentation does not change the company language of saved section names, scope names or notes.

The SHELL stage prices every shell construction system of the estimate building
(footings or raft, ring beams, lintels, sills, plinth, stairs, railings, chimney, roof drainage)
with all matching active services of the estimate price list, using the same rules as
`plan_estimate_element_batch`. Already priced sources and services requiring manual component quantities are
skipped. Unpriced sources of an existing service get a new complementary position; `addedCount` counts actual new
positions. Stage pricing keeps existing rows, quantities and prices. Later model changes correct model quantities of
an editable draft in the background ("The estimate follows the model"); in other estimates review them with
`review_estimate_against_model` and accept them explicitly with `apply_estimate_model_changes`. Every model quantity is priced once, by the same rule as the review's `doubleCounted`: a whole-system
service ("Więźba dachowa — całość (zamiast osobnych pozycji)") and per-member services counting the same timber
("Krokwie", "Murłata", "Jętki") are never added together — the per-member services win when together they cover the
whole system, otherwise the whole-system service is added alone. An existing alternative blocks overlapping services
only on its saved sources, so repeating the call never adds the skipped alternative on those sources. Positions on the default roof framing of a roof without
framing carry the note "Więźba domyślna — skonfiguruj w etapie „Dach i konstrukcja”". In the SHELL stage, `unmatchedElements` with elementType CONSTRUCTION_SYSTEM are systems the price list
cannot price yet — install shell services with
`install_construction_starter_pack` or add them to the price list; `manualElements` need explicit manual quantities. Requires `estimates.write`, a DRAFT estimate and the current modelVersion.

### Price the installations and the finishing at once

The INSTALLATIONS stage prices every installation element of the estimate building with the same rules:
points by kind (sockets, switches, light points, outlets, radiators, boards, manifolds, heat sources, meters, smoke
detectors, ventilation units), circuits by conduit spec with the installation allowance (protections, RCDs, junction
boxes, fittings, underfloor loops), manual routes and risers — one position per service and trade section (electrical,
water, drainage, heating, gas, ventilation). Riser connection points are priced through their risers.

The FINISHING stage prices the closed shell and the finishing: window, door, garage door and roof window
installation by opening style, internal sills, facade layers of external walls, suspended plasterboard ceilings and
attic slopes, plaster of room walls and ceilings, screeds and floor finishes, bathroom tiles and painting — one position
per service and stage section. Nothing is counted twice: room services measure only what build-up layers do not
(`SPACE_WALL_PLASTER_AREA` without lined walls, `SPACE_CEILING_PLASTER_AREA` without suspended ceilings,
`SPACE_SCREED_AREA` only where no screed layer lies under the room — the ground floor slab layers stay with the shell),
wall and slab services price build-up layers only, and saved sources are skipped (e.g. internal sills
priced by the SHELL stage). New matching sources get complementary positions even when the service is already used.
`review_estimate_against_model` lists `uncoveredInstallations` and
`uncoveredFinishing` once the estimate prices an element of the stage; windows drawn later appear in `newElements`.
New automatic FINISHING positions can also propose matching ordinary rooms, including a room split off by
`create_partition`, within their original work section. This does not change the floor-local rules for older or
manually scoped positions. New elements are never added by themselves; accepting them requires
`apply_estimate_model_changes`.
Alternatively, explicitly repeat FINISHING to add missing sources as complementary positions without changing earlier
rows. To deliberately change selected scopes, use `preview_estimate_position_scope` + `assign_estimate_position_scope`.
