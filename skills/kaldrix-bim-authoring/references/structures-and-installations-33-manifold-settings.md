## Manifold settings

Plain installation routes and installation groups are no longer edited through MCP (KDX-177): use circuits
(`create_installation_circuit`, `update_installation_circuit` with `nodeEdit`) for connected cables and pipes; existing
plain routes are listed in `get_floor_detail`. `get_underfloor_manifold_settings` with the `manifoldId` of a
UFH_MANIFOLD point reads its port geometry (`geometry`, `assumed`) and `modelVersion` before editing; read loops with
`list_installation_circuits`.
