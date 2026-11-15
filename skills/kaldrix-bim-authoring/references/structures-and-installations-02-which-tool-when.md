## Which tool when

| The user wants… | Use | Not |
| --- | --- | --- |
| plan all intended trades on one floor from one model snapshot | `plan_floor_installations` — optional `systems` filter, per-trade source → points → connections, canonical actions, next tools | adding an unneeded trade just to complete every guide |
| next step in one trade (source → points → connections) | `get_installation_trade_guide` — current step, done/count per step, tool intents; a gas boiler (`HEAT_SOURCE` with `heatSourceType: GAS_BOILER`) is also the gas appliance | guessing the order or placing a second gas point next to the boiler |
| typical sockets / water / light in a room or whole floor | `add_installation_point_preset` / `add_installation_point_presets_for_floor` | dozens of `add_installation_points` calls |
| one specific point (board, stack, manifold, extra socket) | `add_installation_points` | |
| cables / pipes for the whole floor | `suggest_installation_circuits` → `apply_installation_circuit_suggestions` | `create_installation_circuit` per circuit |
| one hand-picked circuit | `create_installation_circuit` | |
| stack / riser / shaft through floors (drain, water, heating, gas, feeder, ventilation ducts) | `create_installation_riser` (`list_installation_risers`, `update_installation_riser`) | a `DRAIN_STACK` point on every floor and vertical metres by hand |
| heat-recovery ventilation (rekuperacja) | `suggest_ventilation` → `apply_ventilation_suggestions` → `suggest_installation_circuits` `system: "VENTILATION"` | hand-placed diffusers without airflows |
| timber-frame walls (studs, plates, headers) | wall build-up: `create_build_up_from_template` + `assign_build_up`, read `get_wall_frame` | `add_structural_members` stud by stud |
| timber floor / ceiling joists | slab build-up + `get_slab_frame` **or** framing system `TIMBER_FLOOR_JOISTS` — one of them | both on the same spot |
| suspended GK / batten ceiling | framing system `CEILING_GK_SINGLE` / `CEILING_GK_DOUBLE` / `CEILING_TIMBER` | |
| roof framing with battens | framing system `ROOF_RAFTERS` on a roof from `create_roof` | rafters one by one |
| gazebo, carport, cabin skeleton of posts and girts | framing system `TIMBER_POST_FRAME` | `add_post_under_member` per post |
| beam resting on two walls | `add_beam_between_walls` | `add_structural_members` with hand-computed ends |
| post under an existing beam | `add_post_under_member` | |
| any other single member (ring beam, lintel, purlin) | `add_structural_members` (optional `array`) | |
| copy / move / rotate / mirror drawn members | `transform_structural_members` | re-adding members |
| same section, role or height for several drawn members | `update_structural_member` with `memberIds` | one call per member |
| same members or points on another storey | `copy_to_floor` | re-adding them there |
| change spacing, sections, height of a system | `update_framing_system` | `explode_framing_system` |
| change a generated roof member's section, rafter end extension, removal or collar height | `get_framing_members` → `update_framing_member` | `explode_framing_system` |
| edit generated purlins, posts or wall plates | `get_roof_support_plan` → `update_roof_support_plan` | drawing replacements over generated supports |
| unsupported free editing as independent members, explicitly requested by the user | `explode_framing_system` (destructive, loses FRAMING_* quantities) | exploding for an edit already supported parametrically |

Destructive removals — ask the user, then `confirm=true`; need `projects.delete`: `remove_installation_points`, `remove_installation_circuits`, `remove_installation_risers`, `remove_structural_members`, `remove_framing_system`, `explode_framing_system`, `explode_installation_circuit`, `remove_build_up`.

Need `confirm=true` after user approval: `apply_installation_circuit_suggestions`, `update_installation_circuit`, `create_underfloor_heating`, `update_framing_system`, `update_build_up`, `copy_to_floor`, `update_installation_riser`, `update_construction_system`.
Also need `projects.delete`: `update_installation_point`, `update_framing_member`, `transform_structural_members`, `assign_build_up`, `set_building_construction`, `set_foundation_outline`, `set_floor_outline`; `update_installation_riser` when it changes `lines`, `fromFloorId` or `toFloorId`; `update_construction_system` when it changes `params`, `placement` or `stairsEdit`.
