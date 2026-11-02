# storybox

A screenless story box for a toddler. Place a figurine or card with an NFC tag on the lid, it plays. Built on the open source [Marelleo](https://github.com/Zebra-Rodeo/marelleo) design (MIT) by Zebra Rodeo, module version, with a few changes for an under 3 and for a household that does not want a box pulling firmware from the internet.

| | |
|---|---|
| Brain | ESP32-S3-DevKitC-1 N16R8 |
| Reader | PN532 in HSU mode, NTAG213 tags |
| Audio | MAX98357A I2S amp, Visaton FRS 7 4 ohm |
| Storage | microSD, MP3 and M4A, offline only |
| Power | 3000 mAh protected LiPo, Lipo Rider Plus, USB-C |
| Enclosure | Author's STLs printed in PLA on a QIDI Plus5 |

## Changes from upstream

| Change | Why | Where |
|---|---|---|
| OTA update URL blanked | Box never fetches code from a third party server. Updater logs "no update address" and idles | `patches/01-ota-off.patch` |
| Volume cap 21 to 13 | Toddler hearing. Measure and adjust | `patches/02-volume-cap.patch` |
| Lift to pause on | Upstream setting `poseRequise`, toggled from the web page | Runtime setting, no code |
| Tags sealed inside figurines and laminated cards | Choking hazard | Build step |

## Layout

```
docs/PLAN.md            full plan: flags, BOM, wiring, phases, next steps
docs/BOM.md             shopping list with AU sources
docs/WIRING.md          pin table for the module build
docs/BENCH-CHECKLIST.md bring-up day, tick as you go
docs/BUILD-LOG.md       dated notes
patches/                diffs against upstream firmware 0.7.8
firmware/               git submodule, fork of Zebra-Rodeo/marelleo
enclosure/              print settings, STLs live in firmware/boitier
content/                SD card layout example, no media
```

## Build the firmware

```
git clone --recurse-submodules https://github.com/Jedidude2/storybox
cd storybox/firmware
git apply ../patches/01-ota-off.patch ../patches/02-volume-cap.patch
cd firmware
pio run -e boite -t upload
pio run -e boite -t uploadfs
pio device monitor
```

Patches are written against upstream commit at firmware version 0.7.8. If `git apply` fails, upstream moved. Open the patch, the change is one line each.

## Licence

Upstream is MIT. Everything here is MIT too. See `firmware/LICENSE`.
