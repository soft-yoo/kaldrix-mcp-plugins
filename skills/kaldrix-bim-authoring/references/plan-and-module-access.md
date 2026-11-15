# Plan and module access

Use the connected server's current entitlement, module state and scopes. Plans do not grant person permissions.
Existing accounts keep their earlier module matrix; never infer access from a plan name alone. Account type and
hiding a Start section do not change tool access.

- Base: building model, roof geometry, quantities, estimates and customer quotations; for new accounts no
  structures, installations, terrain, schedules or budgets.
- Pro adds construction work, scheduling, budgets, tasks, invoices, time tracking and simple installation points.
- Advanced adds specialist modules: STRUCTURES, INSTALLATION_DESIGN, TERRAIN, GRAPHICS, HISTORY and
  PROJECT_COPY_SHARING (saving a copy from a 3D view link, sharing a project for copying). An owner can disable a
  module included in the plan.
- STRUCTURES: detailed structural authoring, including ROOF_RAFTERS previews and mutations. Basic roof creation and
  the roof takeoff need only project access. Suggest enabling STRUCTURES only if the returned entitlement grants it.
- INSTALLATION_DESIGN (depends on INSTALLATIONS): circuit design, conduit catalog and selection, installation
  planning guides, risers, underfloor heating, optimization and installation automation. Simple points and their
  reads stay in INSTALLATIONS.

After a downgrade, saved geometry, raw element reads, floor rendering and quantity/purchase reports keep existing
model data; project visibility and read scopes still apply. A read report is no permission for a new specialist
design, and tool discovery can still hide tools outside granted modules. Raw saved model: list_elements/get_element.
Saved framing data with project access: get_roof_support_plan, get_framing_members, get_framing_bill_of_materials,
get_framing_drawing, list_structures and get_timber_schedule. Never bypass an unavailable authoring tool through a
generic batch: the backend checks kinds.

If access is denied, explain the server's module/scope decision and direct the owner to Subscription → Features or
Subscription → Compare plans (Polish UI: Subskrypcja → Funkcje, Porównanie planów). Changing an account profile,
adding a menu link or renaming a project cannot grant access. Never promise a price or feature from a static
example; read the live catalog and account entitlement.

When a tool is missing, read list_tool_profiles.unavailableToolGroups: each group names the hidden tools, required
scopes or module, access reason and next step.

| Reason | Remedy |
| --- | --- |
| MODULE_NOT_IN_PLAN | plan review |
| MODULE_DISABLED | owner enables an included feature or dependency |
| MISSING_PERMISSION | person permissions and approved OAuth scopes |
| COMPANY_TYPE | the right company type |
| MODULE_UNAVAILABLE | fresh entitlement and module-state check; do not guess a plan upgrade or feature switch |

If a name is in activeToolNames but absent from the client, refresh the client catalog. Diagnostics do not grant
access: never call a hidden tool to bypass a refusal.
