## Choose medium and components

| Medium | Source → target examples | Authoring choice |
| --- | --- | --- |
| POWER | DISTRIBUTION_BOARD → SOCKET, SOCKET_DOUBLE, SOCKET_WATERPROOF, POWER_SOCKET_400V, OVEN, DISHWASHER, WASHING_MACHINE, DRYER, WATER_HEATER, EV_CHARGER | Separate appliance circuits; allowed cable/protection from catalogue. TREE shares a trunk within one circuit. |
| LIGHTING | DISTRIBUTION_BOARD → LIGHT_POINT_CEILING, LIGHT_POINT_WALL, SWITCH, DETECTOR_CEILING | Create lights first; set SWITCH.controlsPointIds to their IDs. This control relation does not draw or connect a cable. |
| LOW_VOLTAGE | DISTRIBUTION_BOARD → DATA_OUTLET, TV_OUTLET | STAR = separate source-to-endpoint runs. The board point is the supported source, not a detailed patch-panel inventory. |
| FEEDER | DISTRIBUTION_BOARD ↔ ELECTRICAL_RISER or another DISTRIBUTION_BOARD | Only boards and electrical risers are targets. Vertical part: multi-floor riser. |
| COLD_WATER | WATER_METER, WATER_MANIFOLD, WATER_RISER → water fixtures, WATER_MANIFOLD, WATER_RISER or a domestic hot-water source (HEAT_SOURCE with hotWaterSupply, WATER_HEATER) | STAR from a manifold; TREE from meter/riser for an intended shared trunk. The hot-water source and an outgoing riser also need cold supply. |
| HOT_WATER | Domestic hot-water source, compatible WATER_MANIFOLD or hot-water WATER_RISER → hot-water fixtures, WATER_MANIFOLD, WATER_RISER | A fixture may be in both cold and hot circuits; exclude cold-only fixtures. A heating-only boiler or cold-only riser does not supply this medium. |
| CIRCULATION | Catalogue-eligible water source → chosen water endpoints | Create explicitly when requested; cold/hot suggestions do not imply it. Stored network is a tree, not a closed hydraulic ring. |
| DRAIN | DRAIN_STACK → DRAIN_WASHBASIN, DRAIN_SINK, DRAIN_WC, DRAIN_SHOWER, DRAIN_LINEAR, DRAIN_FLOOR_GULLY, DRAIN_APPLIANCE | Slope as a fraction: 0.02 = 2%. Check available depth and fall warnings. Each explicit edge is one pipe. |
| HEATING_SUPPLY_RETURN | HEAT_SOURCE, UFH_MANIFOLD, HEATING_RISER → RADIATOR, UFH_MANIFOLD, HEATING_RISER | Backend counts paired supply/return; never create a second identical circuit to double quantity. |
| GAS | GAS_METER or GAS_RISER → GAS_POINT, GAS_RISER or HEAT_SOURCE with heatSourceType GAS_BOILER | The gas boiler is the same point used for heating. Source and gas-pipe spec from the catalogue; riser across floors. |
| VENT_SUPPLY | MVHR_UNIT → AIR_DISTRIBUTION_BOX; box → SUPPLY_DIFFUSER | Main duct vs flexible branch defaults depend on source. Set diffuser airflow; backend derives flexible parallel runs. |
| VENT_EXTRACT | MVHR_UNIT → AIR_DISTRIBUTION_BOX; box → EXTRACT_DIFFUSER | Keep supply/extract as separate circuits even in one corridor. |
| VENT_INTAKE | MVHR_UNIT → AIR_INTAKE | Dedicated intake duct; check intake/exhaust placement in ventilation guidance. |
| VENT_EXHAUST | MVHR_UNIT → AIR_EXHAUST | Dedicated discharge duct, different from indoor VENT_EXTRACT. |
| UNDERFLOOR_HEATING | UFH_MANIFOLD → derived room fields | Preview parametric SPIRAL or MEANDER loops with `preview_underfloor_heating`, inspect approximate paths/lengths, create with the reviewed version. |

A fixture often needs several model points with distinct IDs, each connected in its own trade: sink = WATER_SINK + DRAIN_SINK; dishwasher = DISHWASHER + WATER_DISHWASHER + DRAIN_APPLIANCE. A point may belong to different media, but not two circuits of the same medium. An electrical CHAIN is cable routing between points, not loads wired in series.
