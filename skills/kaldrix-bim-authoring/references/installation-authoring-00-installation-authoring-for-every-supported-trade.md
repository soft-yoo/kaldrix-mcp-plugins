# Installation authoring for every supported trade

- Every physical write needs the reviewed `expectedModelVersion`; individual element creation also needs `idempotencyKey` (unique and stable per intent; reuse only with identical input after an uncertain response). Example keys below are placeholders, not for reuse across real jobs. After a replay read the live project before another edit.
- Use the WORKSPACE tool profile. Tool availability and schemas come from the runtime tool catalogue, not this guide.
- Modules: simple points need INSTALLATIONS; circuit, riser, HVAC and planning tools need INSTALLATION_DESIGN (depends on INSTALLATIONS); estimates need ESTIMATES. Scopes: projects.read/projects.write as declared; flagged changes also need projects.delete and confirmation. See [plan and module access](plan-and-module-access.md).
- Longer house and estimate playbooks: `kaldrix://projects/structures-and-installations`.
