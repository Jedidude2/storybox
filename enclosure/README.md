# Enclosure

STLs are upstream in `firmware/boitier/`:

| File | Size mm | Notes |
|---|---|---|
| marelleo-corps.stl | 120 x 120 x 80 | Body. Print open face up, no supports |
| marelleo-base.stl | 120 x 120 x 29 | Holds board and cell |
| marelleo-molette.stl | 24 x 24 x 12 | Volume knob. Print this first as the test piece |

## QIDI Plus5 settings for PLA

| Setting | Value |
|---|---|
| Material | PLA, EN 71-3 certified |
| Door and lid | Open |
| Chamber heater | Off |
| Nozzle | 0.4 |
| Layer | 0.2 |
| Walls | 3 |
| Infill | 15 % |
| Supports | None |
| Plate | Textured PEI side, no glue |

Fixings: M3 brass inserts melted in with a soldering iron tip. Never screw straight into PLA.

Reader placement: PN532 antenna flat under the top face, under the engraved circle. 3 mm PLA is fine.

Toddler changes to consider for a second print: fill the base cavity around the cell with a printed cradle so it cannot rattle, recess the USB-C port, keep grille holes under 5 mm.
