## Outline model contract


A project contains independent buildings. Each building has a stable UUID, a name, a description, a type and an outer-face `baseOutline`. Floors belong to one building and inherit this footprint unless they have an `outlineOverride`. Coordinates use project plan metres (x, y); 3D plan coordinates are (x, z), with height on y.


Use the canonical sequence in `kaldrix://projects/house-building-workflow`: project → building → outline → floors → partitions → room labels.

# Room and Opening Templates

Use these values as sensible defaults when the user does not provide exact dimensions.

All dimensions are in metres.

## Room templates

| Space type | Suggested name | Typical width | Typical length | Typical area |
| --- | --- | ---: | ---: | ---: |
| `ROOM` | Living room | 4.5 | 5.5 | 24 |
| `ROOM` | Bedroom | 3.0 | 4.0 | 12 |
| `ROOM` | Home office | 2.5 | 3.0 | 8 |
| `BATHROOM` | Bathroom | 2.0 | 2.5 | 5 |
| `BATHROOM` | Toilet | 1.2 | 1.5 | 2 |
| `KITCHEN` | Kitchen | 3.0 | 4.0 | 12 |
| `CORRIDOR` | Corridor | 1.2 | 5.0 | - |
| `STORAGE` | Wardrobe | 1.5 | 2.0 | 3 |
| `TECHNICAL` | Technical room | 2.0 | 2.5 | 5 |
| `GARAGE` | Single-car garage | 3.0 | 5.5 | 16 |
| `STAIRCASE` | Staircase | 2.5 | 4.0 | 10 |

## Opening templates

| Opening type | Suggested name | Width | Height | Sill height |
| --- | --- | ---: | ---: | ---: |
| `DOOR` | Interior door | 0.9 | 2.0 | 0.0 |
| `DOOR` | Bathroom door | 0.8 | 2.0 | 0.0 |
| `DOOR` | Entrance door | 1.0 | 2.1 | 0.0 |
| `WINDOW` | Standard window | 1.4 | 1.4 | 0.85 |
| `WINDOW` | Large window | 1.8 | 1.6 | 0.75 |
| `WINDOW` | Bathroom window | 0.6 | 0.6 | 1.5 |

## Placement notes

- Doors usually use `sillHeight = 0.0`.
- Typical windows use `sillHeight = 0.80-0.90`.
- Bathroom windows are often smaller and higher.
- Always validate that `positionOnWall + width <= wall.length`.
- Always validate that `sillHeight + height <= wall.height`.
