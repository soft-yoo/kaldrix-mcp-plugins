# Plan and module access

Use the connected server's current entitlement, module state and scopes. Plans do not grant
person permissions. Existing accounts retain their earlier module matrix; never infer their
access from a plan name alone. Account type and hiding a Start section do not change tool access.

- Base includes the building model, roof geometry, quantities, estimates and customer quotations.
  It excludes structures, installations, terrain, schedules and budgets for new accounts.
- Pro adds construction work, scheduling, budgets, tasks, invoices, time tracking and simple installation points.
- Advanced adds specialist modules, including STRUCTURES and INSTALLATION_DESIGN, as well as
  TERRAIN, GRAPHICS, HISTORY and PROJECT_COPY_SHARING (saving a copy from a 3D view link, sharing a project for
  copying). An owner can disable a module included in the account's plan.
- Detailed structural authoring, including ROOF_RAFTERS previews and mutations, requires STRUCTURES.
  Basic roof creation and the roof takeoff remain available with project access. Do not suggest
  enabling STRUCTURES unless the account's returned entitlement actually grants it.
- Circuit design, conduit catalog and selection, installation planning guides, risers, underfloor
  heating, optimization and installation automation require INSTALLATION_DESIGN. It depends on
  INSTALLATIONS. Simple points and their read operations remain in INSTALLATIONS.

Saved geometry, raw element reads, floor rendering and quantity/purchase reports retain existing
model data after a downgrade. The user's project visibility and read scopes still apply. A read
report is not permission to create a new specialist design; tool discovery can still hide tools
outside the granted modules. Use list_elements/get_element for raw saved model inspection.
For framing, get_roof_support_plan, get_framing_members, get_framing_bill_of_materials,
get_framing_drawing, list_structures and get_timber_schedule use project access for saved data.
Do not bypass an unavailable authoring tool through a generic batch: the backend checks kinds.

If access is denied, explain the server's module/scope decision and direct the owner to
Subscription → Features or Subscription → Compare plans (Polish UI: Subskrypcja → Funkcje,
Porównanie planów). Changing an account profile, adding a menu link or
renaming a project cannot grant access. Do not promise a price or a feature from a static example;
read the live catalog and account entitlement.

When a tool is missing, read list_tool_profiles.unavailableToolGroups. Each group names the
hidden tools, required scopes or module, access reason and next step. MODULE_NOT_IN_PLAN
needs plan review; MODULE_DISABLED needs the owner to enable an included feature or dependency.
MISSING_PERMISSION needs person permissions and approved OAuth scopes. COMPANY_TYPE needs
the right company type. MODULE_UNAVAILABLE needs a fresh entitlement and module-state check;
do not guess that a plan upgrade or feature switch is the remedy. If a name is in activeToolNames but absent from the client, refresh the
client catalog. Diagnostics do not grant access: never call a hidden tool to bypass a refusal.
