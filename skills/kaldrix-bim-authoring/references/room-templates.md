# Room and Opening Templates

Sensible defaults (metres) when the user gives no exact dimensions. Model contract and canonical sequence:
`kaldrix://projects/house-building-workflow`.

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

- Doors usually use `sillHeight = 0.0`; typical windows `sillHeight = 0.80-0.90`; bathroom windows are often smaller and higher.
- Always validate that `positionOnWall + width <= wall.length` and that `sillHeight + height <= wall.height`.
