## Discover before writing

Circuit, riser and planning tools need INSTALLATION_DESIGN; simple points use INSTALLATIONS ([plan and module access](plan-and-module-access.md)).

1. Read `get_project_detail`, then `plan_floor_installations(projectId, floorId, expectedModelVersion)`: one snapshot of all six trade guides; `systems` limits it to selected trades. An unconfigured trade is optional: add gas or another installation only if requested. Follow SOURCE → POINTS → CONNECT and replan after saving. Read `get_floor_detail`, `list_installation_points`, `list_installation_circuits`, `list_installation_risers`; reuse existing equipment and endpoints.
2. `kaldrix://projects/installation-point-kinds` lists every supported component with trade, mount and height. Use `add_installation_points` for individual equipment/endpoints; presets are only typical room sets and miss many kinds.
3. `get_installation_conduit_catalog` is authoritative for media, compatible specs, source/target restrictions, source-dependent defaults, routing bounds and 3D capabilities. Never guess compatibility from names; `targetKinds=null` means general trade validation, not permission to mix trades.
4. `get_installation_circuit_selection` (current expectedModelVersion + medium) gives real source/target IDs. Filter eligible targets by room membership (one or several rooms, same floor). Typical targets are suggestions; other eligible choices stay possible.
5. Automatic grouping: `suggest_installation_circuits` → `apply_installation_circuit_suggestions` with the exact reviewed modelVersion, system and language (language sets generated names in preview and saved circuits; default pl; use the project's language). Chosen connections: `create_installation_circuit` with explicit sourcePointId, pointIds and the selection's modelVersion.
6. Inspect `get_installation_circuit_layout`, `validate_floor_geometry` and trade reports. A saved circuit with warnings does not prove a completed installation; report missing sources, unreachable points, crossings and unconnected endpoints.

After each write use the returned IDs and new modelVersion. MODEL_VERSION_CONFLICT requires a fresh read and review; never substitute a newer version into a previously reviewed proposal. In the JSON below replace every `<...>` placeholder with IDs/versions from real reads; snippets illustrate arguments, not executable UUIDs.
