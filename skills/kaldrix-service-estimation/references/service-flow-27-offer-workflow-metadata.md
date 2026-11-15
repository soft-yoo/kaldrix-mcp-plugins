## Offer workflow metadata

`get_estimate_offer_workflow` (`estimateId`, read-only; sends no email, shares no link) returns server-owned
action availability, form limits and validity preview (same calendar rule as PDF generation and actual send;
sending recalculates from its execution time). `latestOffer` has recipient e-mail and client answer details
only with `customers.read`; the client link (`shareUrl`) is never returned. Respect `canSend`, `canResend`,
`canRevoke` and `sendUnavailableReason` (decided by enabled modules, the person's permissions and document
state). Without `estimates.write` every write action is disabled with `sendUnavailableReason`
`WRITE_SCOPE_REQUIRED`, even if the person has that permission in the app.

### Project workflow

`get_project_workflow` (`projectId`; also `ProjectDto.workflow` and `projectWorkflow` in GraphQL) returns the
backend-selected stage and ordered steps. Follow `nextStepId` only when present; use `actionEnabled`, not the
person's permissions or document status. Missing modules hide their steps; read failures remain errors. OAuth
scopes limit the result: `projects.read` alone gives project steps only; `estimates.read`, `customers.read`,
`construction-log.read` and `invoices.read` add their domain facts. Actions need the matching write scope,
granted to the connector and held by the person. Read-only: creates no estimate, sends no offer, keeps the model.

### After the offer is accepted

The accepted estimate becomes primary and the contract plan: prices and contract quantities freeze and
`update_estimate_sheet` accepts only executed quantities (`actualQuantity` or `clearActualQuantity`). They drive
progress of the construction stages made from estimate sections (`get_project_schedule`) and the contract's
stage instalments. The contract comes from the accepted offer (`list_project_contracts`,
`create_project_contract`, `get_project_contract`; one per accepted offer, several per project; default
instalments 20% at signing, 70% after the estimate-section stages, 10% after final handover). Start of works,
stages, acceptance protocols and completion: `kaldrix://mcp/agent-workflow`, section "From the accepted offer to
the keys". A renovation without a drawing (`workType=RENOVATION`) uses manual quantities and the same process.

### Bulk service action metadata

`plan_estimate_element_batch` (read-only): one element, visible active catalog recipe IDs (1 to 2000 distinct)
and the current modelVersion. Follow `actionEnabled`, pending recipe IDs and `maxSelection`; never recalculate
duplicates or quantities in the client. Services with manual components come back as `manualRecipeIds`; add
them separately with explicit manual quantities. Archived price lists and non-draft estimates disable the
action; without `estimates.write` it returns WRITE_SCOPE_REQUIRED. A plan is no authorization to write; actual
operations validate current quantities and permissions.

### Price the model by stages at once

`price_estimate_from_model` (`estimates.write`, DRAFT estimate, current modelVersion) runs SHELL, FINISHING and
INSTALLATIONS (all by default, or those in `stages`) in that order; per stage: PRICED, NOTHING_TO_PRICE (no
elements of that stage), FAILED with `errorCode` (earlier stages stay saved) or NOT_RUN. PRICED means processed:
it may add zero positions and still report unmatched or manual elements, so inspect those lists and
`get_estimate_checks` before reporting full coverage. English MCP presentation keeps the company language of
saved section names, scope names and notes.

All stages use the matching active services of the estimate price list with the rules of
`plan_estimate_element_batch`; already priced sources and services needing manual component quantities are
skipped. Unpriced sources of an already used service get a new complementary position (`addedCount` counts
actual new positions); existing rows, quantities and prices stay. Later model changes correct a draft in the
background ("The estimate follows the model"); other estimates: `review_estimate_against_model`, then explicit
`apply_estimate_model_changes` with the review's `modelVersion` as required `expectedModelVersion`.

**SHELL** prices every shell construction system of the estimate building (footings or raft, ring beams,
lintels, sills, plinth, stairs, railings, chimney, roof drainage). Each model quantity is priced once, by the
review's `doubleCounted` rule: a whole-system service ("Więźba dachowa — całość (zamiast osobnych pozycji)") and
per-member services counting the same timber ("Krokwie", "Murłata", "Jętki") are never added together — the
per-member services win when together they cover the whole system, else the whole-system service is added
alone. An existing alternative blocks overlapping services only on its saved sources, so repeating the call
never adds the skipped alternative there. Positions on the default framing of a roof without framing carry the
note "Więźba domyślna — skonfiguruj w etapie „Dach i konstrukcja”". `unmatchedElements` with elementType
CONSTRUCTION_SYSTEM are systems the price list cannot price yet — install shell services with
`install_construction_starter_pack` or add them to the price list; `manualElements` need explicit manual quantities.

### Price the installations and the finishing at once

**INSTALLATIONS** prices every installation element of the building: points by kind (sockets, switches, light
points, outlets, radiators, boards, manifolds, heat sources, meters, smoke detectors, ventilation units),
circuits by conduit spec with the installation allowance (protections, RCDs, junction boxes, fittings,
underfloor loops), manual routes and risers — one position per service and trade section (electrical, water,
drainage, heating, gas, ventilation). Riser connection points are priced through their risers.

**FINISHING** prices the closed shell and finishing: window, door, garage door and roof window installation by
opening style, internal sills, facade layers of external walls, suspended plasterboard ceilings and attic
slopes, plaster of room walls and ceilings, screeds and floor finishes, bathroom tiles and painting — one
position per service and stage section. Nothing counts twice: room services measure only what build-up layers
do not (`SPACE_WALL_PLASTER_AREA` without lined walls, `SPACE_CEILING_PLASTER_AREA` without suspended ceilings,
`SPACE_SCREED_AREA` only where no screed layer lies under the room — ground floor slab layers stay with the
shell); wall and slab services price build-up layers only; saved sources are skipped (e.g. internal sills
priced by SHELL). New matching sources get complementary positions even for a used service.
`review_estimate_against_model` lists `uncoveredInstallations` and `uncoveredFinishing` once the estimate prices
an element of that stage; windows drawn later appear in `newElements`. New automatic FINISHING positions may
also propose matching ordinary rooms, including one split off by `create_partition`, within their original work
section; floor-local rules of older or manually scoped positions stay. New elements are never added by
themselves: accept them with `apply_estimate_model_changes`, or explicitly repeat FINISHING to add missing
sources as complementary positions without changing earlier rows. To deliberately change selected scopes:
`preview_estimate_position_scope` + `assign_estimate_position_scope`.
