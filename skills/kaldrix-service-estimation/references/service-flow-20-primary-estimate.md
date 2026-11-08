## Primary Estimate

A project can have many estimates (variants, a copy of an approved one); exactly one active estimate is primary
(`isPrimary`, called "Aktywny" in the estimate list and "Kosztorys główny" in the project settlement tab). The first
estimate of a project becomes primary automatically. `set_primary_project_estimate` moves the flag and clears it on
the others; its status does not matter. `deactivate_project_estimate` on the primary estimate hands the flag to the
most recently created remaining estimate. Changing the primary estimate changes the plan in every settlement figure,
so confirm the choice with the user first.
