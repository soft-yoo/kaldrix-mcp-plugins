## Choose medium and components

| Medium | Individual source and target examples | Authoring choice |
| --- | --- | --- |
| POWER | DISTRIBUTION_BOARD → SOCKET, SOCKET_DOUBLE, SOCKET_WATERPROOF, POWER_SOCKET_400V, OVEN, DISHWASHER, WASHING_MACHINE, DRYER, WATER_HEATER, EV_CHARGER | Separate appliance circuits; choose allowed cable/protection from catalogue. TREE shares a trunk within one circuit. |
| LIGHTING | DISTRIBUTION_BOARD → LIGHT_POINT_CEILING, LIGHT_POINT_WALL, SWITCH, DETECTOR_CEILING | Create lights first; set SWITCH.controlsPointIds to those IDs. This control relation does not itself draw or connect a cable. |
| LOW_VOLTAGE | DISTRIBUTION_BOARD → DATA_OUTLET, TV_OUTLET | STAR gives separate source-to-endpoint runs. The board point is the model's supported source; it does not represent a detailed patch-panel inventory. |
| FEEDER | DISTRIBUTION_BOARD ↔ ELECTRICAL_RISER or another DISTRIBUTION_BOARD | Only boards and electrical risers are targets. Use a multi-floor riser for the vertical part. |
| COLD_WATER | WATER_METER, WATER_MANIFOLD, WATER_RISER → water fixtures, WATER_MANIFOLD, WATER_RISER or a domestic hot-water source (HEAT_SOURCE with hotWaterSupply, WATER_HEATER) | STAR from a manifold; TREE from meter/riser when sharing a trunk is intended. The hot-water source and an outgoing riser also need cold supply. |
| HOT_WATER | A domestic hot-water source, compatible WATER_MANIFOLD or hot-water WATER_RISER → hot-water fixtures, WATER_MANIFOLD, WATER_RISER | A fixture may belong to both cold and hot circuits. Exclude cold-only fixtures from the hot circuit. A heating-only boiler and a cold-only riser do not supply this medium. |
| CIRCULATION | Catalogue-eligible water source → chosen water endpoints | Create explicitly when requested; automatic cold/hot suggestions do not imply a circulation loop. The stored network is a tree, not a closed hydraulic ring. |
| DRAIN | DRAIN_STACK → DRAIN_WASHBASIN, DRAIN_SINK, DRAIN_WC, DRAIN_SHOWER, DRAIN_LINEAR, DRAIN_FLOOR_GULLY, DRAIN_APPLIANCE | Set slope as a fraction: 0.02 = 2%. Check available depth and fall warnings. Each explicit edge is one pipe. |
| HEATING_SUPPLY_RETURN | HEAT_SOURCE, UFH_MANIFOLD, HEATING_RISER → RADIATOR, UFH_MANIFOLD, HEATING_RISER | Paired supply/return is counted by the backend. Do not create a second identical circuit to double the quantity. |
| GAS | GAS_METER or GAS_RISER → GAS_POINT, GAS_RISER or HEAT_SOURCE with heatSourceType GAS_BOILER | The gas boiler is the same point used for heating. Source and compatible gas-pipe spec come from the catalogue; use a riser across floors. |
| VENT_SUPPLY | MVHR_UNIT → AIR_DISTRIBUTION_BOX; box → SUPPLY_DIFFUSER | Main duct versus flexible branch defaults depend on source. Set diffuser airflow; backend derives flexible parallel runs. |
| VENT_EXTRACT | MVHR_UNIT → AIR_DISTRIBUTION_BOX; box → EXTRACT_DIFFUSER | Keep supply/extract as separate circuits even when they occupy one corridor. |
| VENT_INTAKE | MVHR_UNIT → AIR_INTAKE | Dedicated intake duct; check intake/exhaust placement in ventilation guidance. |
| VENT_EXHAUST | MVHR_UNIT → AIR_EXHAUST | Dedicated discharge duct; this is different from indoor VENT_EXTRACT. |

| UNDERFLOOR_HEATING | UFH_MANIFOLD → derived room fields | Preview parametric SPIRAL or MEANDER loops with `preview_underfloor_heating`, inspect approximate paths and lengths, then create with the reviewed version. |

A fixture often needs several separate model points: a sink can have WATER_SINK and DRAIN_SINK; a dishwasher can have DISHWASHER, WATER_DISHWASHER and DRAIN_APPLIANCE. Create these with distinct IDs and connect each to its own trade. A point may belong to different media, but not two circuits of the same medium. An electrical CHAIN describes cable routing between points, not loads electrically wired in series.
