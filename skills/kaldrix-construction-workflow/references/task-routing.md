## Choose the task before choosing a write

When a matching plugin skill is installed, use `kaldrix-bim-authoring` for model,
plan, structure or installation topics, and `kaldrix-service-estimation` for
price-list or estimate changes. Choose the topic from its index. The URI column
below is the complete fallback for clients without those skills or for resolving
conflicting guidance; do not fetch every full guide before an operation.

| User intent | Start with | Focused reference |
| --- | --- | --- |
| Create a new house and initial quote | Read the required inputs of `create_quick_house` and the company price-list workflow | `kaldrix://estimation/service-flow` |
| Inspect or change an existing project | `get_projects` → `get_project_detail`; select the existing building and floor IDs | `kaldrix://projects/house-building-workflow` |
| Add devices, endpoints and installation routes | `plan_floor_installations` (INSTALLATION_DESIGN) at the reviewed version, with only the requested trades; points alone need INSTALLATIONS | `kaldrix://projects/installation-authoring` |
| Edit roof framing or determine timber quantities | `get_framing_members` / `get_framing_bill_of_materials` for the system; `get_timber_schedule` for all project timber | `kaldrix://projects/structures-and-installations` |
| Price an existing model or review a saved estimate | `list_project_estimates`; price unpriced work with `price_estimate_from_model`, review existing rows with `review_estimate_against_model` | `kaldrix://estimation/service-flow` |
| Go back after a mistake, or save the state before a large change | `list_project_versions` → `preview_project_version`; `restore_project_version` only after the user approves the preview; `save_project_version` before a risky change | `kaldrix://projects/house-building-workflow` |
| Store a requested deliverable or attach an existing file | Read the document policy and upload constraints, then choose the supported upload flow | `kaldrix://documents/agent-documentation-policy` |
| Run the works after the offer: contract, start, stages, log, acceptance, handover | `get_project_workflow` (phase and next step), then `list_project_contracts` / `get_project_realization` | section "From the accepted offer to the keys" below |
| Track an investor's own construction money | `get_project_budget` | section "Private investor accounts" below |

These are entry points, not a list of required mutations. An inspection request does not authorize creating a new
project, replacing a manual route or refreshing frozen prices. Read only the references needed for the task.

IDs are not interchangeable: projectId identifies the project, buildingId its building, floorId that building's
floor, and estimateId a saved estimate. Obtain an estimateId from `list_project_estimates`; `get_project_estimate`
takes estimateId, not projectId. For an estimate position, use the position/serviceOccurrenceId returned by its
rows, not a recipeId from the company catalogue. Follow each tool's exact input schema.

Geometry dimensions use metres; plan coordinates are x/y, 3D height is y and plan depth is z. Financial amounts
are net or gross as named; VAT values use the fraction specified by the tool (0.23 means 23%). Quantity rules
define their units. Distinguish material metres, physical endpoint distance and timber stock pieces; the domain
guides explain their scopes. Do not add reserve or paired/parallel multipliers to already computed quantities.
