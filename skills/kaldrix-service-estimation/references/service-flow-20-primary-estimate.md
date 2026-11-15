## Primary Estimate

A project can have many estimates (variants, a copy of an approved one); exactly one active estimate is
primary (`isPrimary`; "Aktywny" in the estimate list, "Kosztorys główny" in the project settlement tab). The
first estimate of a project becomes primary automatically. `set_primary_project_estimate` moves the flag and
clears it on the others, regardless of status. `deactivate_project_estimate` on the primary hands the flag to
the most recently created remaining estimate. Changing the primary estimate changes the plan in every
settlement figure; confirm the choice with the user first.
