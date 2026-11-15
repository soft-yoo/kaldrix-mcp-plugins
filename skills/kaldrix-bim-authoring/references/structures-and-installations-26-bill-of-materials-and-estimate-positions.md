## Bill of materials and estimate positions

1. `get_framing_bill_of_materials` (one `systemId` or whole floor) returns lines with `quantityRule`, English `label`,
   `quantity`, `unit`, plus aggregates.
   - Timber order: `get_timber_schedule` (`projectId`) — timber pieces of framing systems, plan-drawn members and wall /
     floor frames grouped by role, section and length, totals per section and a cut list into 4 / 5 / 6 m stock lengths
     (smallest purchase, 5 mm saw kerf, battens excluded); same schedule as the project report. Rows group lengths to
     the nearest centimetre.
   - Its `totalVolume` includes all timber incl. covering timber (battens, counter battens, fascia and barge boards);
     the framing BOM's `FRAMING_TIMBER_VOLUME` excludes covering timber. Different scopes — never add them together.
   - Inspect every `cutList[].oversize`: those pieces do not fit available stock and are excluded from `bars` and
     purchased `totalLength` (even `bars: 0` can mean a large oversize order). Report their lengths and counts
     separately; order to size or change the model to use a supported joint. Never silently split a rafter. Wall frames
     with aggregate role quantities have no individual lengths and are outside this cut list.
   - Edit a saved roof member: `get_framing_members` → stable `memberId` + `modelVersion` → `update_framing_member`,
     then re-read members and `get_timber_schedule`. A section override must appear as a separate schedule section;
     never reuse the old report after editing.
2. Each line maps to a service recipe on the matching element type and rule:
   - points: `INSTALLATION_POINT` + `INSTALLATION_POINT_COUNT` + `PCS` with `elementSubtypes: ["SOCKET","SOCKET_DOUBLE"]`
     (InstallationPointKind names);
   - framing: `FRAMING_SYSTEM` + `FRAMING_AREA` / `FRAMING_BOARD_AREA` (`M2`), `FRAMING_MAIN_LENGTH` /
     `FRAMING_SECONDARY_LENGTH` / `FRAMING_PERIMETER_LENGTH` (`MB`), `FRAMING_FIXING_COUNT` / `FRAMING_CONNECTOR_COUNT` /
     `FRAMING_MEMBER_COUNT` (`PCS`), `FRAMING_TIMBER_VOLUME` (`M3`), `elementSubtypes` = FramingSystemKind names (e.g.
     `["CEILING_GK_SINGLE","CEILING_GK_DOUBLE"]`);
   - members: `STRUCTURAL_MEMBER` + `STRUCTURAL_MEMBER_LENGTH` (`MB`) / `STRUCTURAL_MEMBER_VOLUME` (`M3`) /
     `STRUCTURAL_MEMBER_COUNT` (`PCS`), `elementSubtypes` = StructuralMemberRole names (e.g. `["RING_BEAM"]`).
3. Find a service with `list_service_recipes` or create one with `create_service_recipe`.
   `install_construction_starter_pack` adds ready services to a price list (idempotent, existing prices untouched),
   e.g. "Montaż gniazda 2P+Z", "Sufit GK — montaż z płytami", "Profil CD60", "Krokwie — drewno C24" and build-up layer
   services ("Płyta GK na ścianie — montaż", "Wełna mineralna 15 cm w szkielecie ściany", "Mur z pustaka ceramicznego 25 — pustak, zaprawa i murowanie"
   for external and load-bearing walls, "Ścianka działowa z pustaka ceramicznego 11,5 — …" for PARTITION walls, "Strop
   żelbetowy monolityczny — beton z pompą, zbrojenie i deskowanie (m³)", "Dachówka — krycie").
4. `list_estimate_quantity_sources` for the service and project, then `add_catalog_service_to_estimate` with the
   returned scopes (one frozen row per scope: a point, a member or a system).
