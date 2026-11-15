## Choose the task before choosing a write

With the plugin skills installed, use the topic index of `kaldrix-bim-authoring` (model, plan, structure,
installations) or `kaldrix-service-estimation` (price lists, estimates). The URI column is the fallback without them
or for conflicting guidance; never fetch every full guide before an operation.

| User intent | Start with | Focused reference |
| --- | --- | --- |
| Create a new house and initial quote | Required inputs of `create_quick_house` and the company price-list workflow | `kaldrix://estimation/service-flow` |
| Inspect or change an existing project | `get_projects` → `get_project_detail`; select existing building and floor IDs | `kaldrix://projects/house-building-workflow` |
| Add devices, endpoints and installation routes | `plan_floor_installations` (INSTALLATION_DESIGN) at the reviewed version, only requested trades; points alone need INSTALLATIONS | `kaldrix://projects/installation-authoring` |
| Edit roof framing or determine timber quantities | `get_framing_members` / `get_framing_bill_of_materials` per system; `get_timber_schedule` for all project timber | `kaldrix://projects/structures-and-installations` |
| Price an existing model or review a saved estimate | `list_project_estimates`; `price_estimate_from_model` for unpriced work, `review_estimate_against_model` for existing rows | `kaldrix://estimation/service-flow` |
| Go back after a mistake, or save the state before a large change | `list_project_versions` → `preview_project_version`; `restore_project_version` only after the user approves the preview; `save_project_version` before a risky change | `kaldrix://projects/house-building-workflow` |
| Store a requested deliverable or attach an existing file | Document policy and upload constraints, then the supported upload flow | `kaldrix://documents/agent-documentation-policy` |
| Run the works after the offer: contract, start, stages, log, acceptance, handover | `get_project_workflow` (phase, next step), then `list_project_contracts` / `get_project_realization` | section "From the accepted offer to the keys" |
| Track an investor's own construction money | `get_project_budget` | section "Private investor accounts" |

These are entry points, not required mutations. Inspection does not authorize creating a project, replacing a manual
route or refreshing frozen prices.

IDs are not interchangeable: projectId (project), buildingId (its building), floorId (that building's floor),
estimateId (a saved estimate, from `list_project_estimates`; `get_project_estimate` takes estimateId, not projectId).
For an estimate position use the position/serviceOccurrenceId from its rows, not a catalogue recipeId. Follow each
tool's exact input schema.

Amounts are net or gross as named; VAT is the tool's fraction (0.23 = 23%). Quantity rules define their units. Distinguish material metres, physical endpoint
distance and timber stock pieces (domain guides explain scopes). Never add reserve or paired/parallel multipliers to
already computed quantities.
