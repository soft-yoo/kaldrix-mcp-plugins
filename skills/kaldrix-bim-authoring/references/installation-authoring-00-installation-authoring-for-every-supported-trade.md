# Installation authoring for every supported trade

Every physical write requires the reviewed `expectedModelVersion`. Individual element creation also requires `idempotencyKey`: use a unique stable key for one intent and reuse it only with identical input after an uncertain response. Example keys below are placeholders for independent intents; do not reuse them across real jobs. After a replay read the live project before another edit.


Use this workflow with the WORKSPACE tool profile. Tool availability and parameter schemas come from the runtime tool catalogue, not this guide. Simple points require INSTALLATIONS; circuit, riser, HVAC and planning tools require INSTALLATION_DESIGN, which depends on INSTALLATIONS. Reads and writes need their declared projects.read/projects.write scopes, and flagged changes also need projects.delete and confirmation; estimates require ESTIMATES. Check [plan and module access](plan-and-module-access.md), then read `kaldrix://projects/structures-and-installations` for the longer house and estimate playbooks.
