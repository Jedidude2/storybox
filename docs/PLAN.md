# Story Box build plan (Marelleo clone, toddler edition)

Decisions locked in this session

| Item | Decision |
|---|---|
| Platform | ESP32-S3 module build, Marelleo firmware built from source (`pio run -e boite`) |
| Triggers | NFC figurines and NFC cards, both NTAG213 |
| Content | Local microSD only, MP3 and M4A. No radio, no podcasts |
| Power | Single 3000 mAh protected LiPo cell, USB-C charging via Lipo Rider Plus |
| Age | Under 3 |
| Parent features | Lift to pause (built in), hard volume cap (two layers), see "Suggested extras" |
| Enclosure | Self printed on a QIDI Plus5 (about $1,069 AUD) using the author's STLs. PLA, door open, chamber heater off |
| Docs home | New GitHub repo, this file is the seed |

Sources: github.com/Zebra-Rodeo/marelleo (MIT, firmware 0.7.8 at time of writing), firmware/src/main.cpp, firmware/src/version.h, firmware/src/library.h, carte/schema.md.

---

## 1. Flags first

| Flag | Why | What to do |
|---|---|---|
| **OTA from a third party, hard coded** | `version.h` sets `MAJ_SOURCE_DEFAUT` to `https://maj.marelleo.org/marelleo-proto.json`. Box pulls and flashes a binary from the author's server. Integrity check is MD5 and size only, no signature. Tries 3 boots then rolls back if Wi-Fi fails, which does not catch a malicious but working image. | Build from source and set `MAJ_SOURCE_DEFAUT` to `""` or to your own GitHub Pages manifest. Decide before first flash. |
| **Config web page has no auth** | `http://marelleo.local` lets anyone on the LAN upload files and rewrite the library. | Put the box on the IoT VLAN or guest SSID. Wi-Fi is only needed for content upload, so leaving it off the network between uploads is also fine. |
| **LiPo in a toddler's room** | Lipo Rider Plus charges at up to 1 A and the author measured the charger running hot. | Use a protected cell with JST-PH 2.0 lead, 2000 to 3000 mAh, from a reputable AU supplier. Cell sits inside the sealed base, never user accessible. Charge supervised, not overnight in the cot room. |
| **NFC stickers are a choking and swallow hazard** | 25 mm NTAG213 sticker on the underside of a toy peels off. | Figurines: embed the tag inside a cavity and seal with epoxy, or glue a felt pad over it. Cards: laminate the tag inside the card. Never a bare sticker. |
| **Firmware pinout is hard coded** | Wrong env or wrong wiring gives silent failure. | Use the `boite` env with the DevKitC-1 N16R8 exactly as listed. Do not substitute an N8R2 board. |
| **PN532 TX/RX crossover** | Module labelled SDA/TXD goes to ESP RX (GPIO 8), SCL/RXD goes to ESP TX (GPIO 9). RX to RX powers up and never answers. | Wire crossed and set the two DIP switches to HSU (printed on the back of the red board). |

---

## 2. Bill of materials

Prices are rough AUD, Oct 2026. Check Core Electronics, Little Bird, Jaycar, and AliExpress for the modules.

| # | Part | Qty | Notes | AUD |
|---|---|---|---|---|
| 1 | ESP32-S3-DevKitC-1 **N16R8** (16 MB flash, 8 MB octal PSRAM) | 1 | Must be N16R8. CH343 USB serial variant preferred, it survives crashes for `pio device monitor` | 25 |
| 2 | PN532 NFC module, red board V3 | 1 | HSU mode via DIP switches | 15 |
| 3 | MAX98357A I2S amp breakout | 1 | Adafruit or clone. Gain pin matters, see section 5 | 12 |
| 4 | Visaton FRS 7, 4 ohm | 1 | 6.5 cm full range. Any 4 ohm 3 W speaker of similar size works | 25 |
| 5 | microSD breakout, 3.3 V with level shifter | 1 | Author used one with a 74LVC125 buffer. Catalex style module is fine | 8 |
| 6 | microSD card 8 to 32 GB, FAT32 | 1 | SanDisk or Samsung, not no-name | 15 |
| 7 | Seeed Lipo Rider Plus | 1 | USB-C in, 5 V out, charges cell | 20 |
| 8 | LiPo cell, protected, JST-PH 2.0, 3000 mAh | 1 | Core Electronics or Little Bird stock these. Locked in | 28 |
| 9 | 10 k potentiometer **with switch** | 1 | Switch cuts battery, that is the power button | 5 |
| 10 | Momentary push button | 1 | Pause / next. Big arcade style for toddler hands | 3 |
| 11 | NTAG213 25 mm **on-metal** stickers | 20 | Author's recommendation, works on plain toys too | 15 |
| 12 | Blank NTAG213 PVC cards | 10 | For the card set | 10 |
| 13 | Perfboard, header strips, Dupont wire, JST-PH leads | 1 | Breadboard first, perfboard once working | 15 |
| 14 | M3 brass heat-set inserts, M3 screws | 10 | If going with a printed case | 8 |
| | **Total before enclosure** | | | **~200** |

---

## 3. Wiring (module build, `boite` env)

| ESP32-S3 GPIO | Goes to | Note |
|---|---|---|
| 4 | MAX98357A SD | Mutes idle hiss |
| 5 | MAX98357A BCLK | |
| 6 | MAX98357A LRC | |
| 7 | MAX98357A DIN | |
| 8 | PN532 SDA/TXD | ESP RX |
| 9 | PN532 SCL/RXD | ESP TX |
| 10 | Pot wiper | Must be ADC1, GPIO 1 to 10 |
| 11 | SD CS | |
| 12 | SD SCK | |
| 13 | SD MOSI | |
| 14 | SD MISO | |
| 18 | Button to GND | Internal pull-up, no resistor |
| 48 | Onboard LED | Already on the DevKit |
| 3V3 | PN532 VCC, SD module VCC | |
| 5V (VIN) | MAX98357A VIN | From Lipo Rider 5 V out |
| GND | Everything | Common ground |

Power chain: cell -> Lipo Rider Plus battery JST -> Lipo Rider 5 V out -> pot switch -> ESP 5V pin and amp VIN. USB-C on the Lipo Rider charges the cell.

Pot outer pins: one to 3V3, one to GND, wiper to GPIO 10. Speaker: amp + and - to the two speaker terminals, polarity does not matter for one speaker.

Free GPIO: 15, 16, 17 if you add anything.

---

## 4. Software setup

1. Install VS Code and the PlatformIO extension.
2. `git clone https://github.com/Zebra-Rodeo/marelleo` then `cd marelleo/firmware`.
3. Edit `src/version.h`, set `MAJ_SOURCE_DEFAUT` for the non `CARTE_IMPRIMEE` branch to `""` (or your own manifest URL).
4. Edit `src/main.cpp` line with `VOLUME_MAX = 21` to your cap (see section 5).
5. `pio run -e boite -t upload` with the DevKit on USB.
6. `pio run -e boite -t uploadfs` once to seed LittleFS (holds Wi-Fi config and fallback test stories).
7. `pio device monitor` at 115200 and watch for SD mount, PN532 answer, volume line.
8. First boot opens AP `Marelleo-XXXX`. Join from a phone, pick home Wi-Fi (IoT SSID).
9. Browse to `http://marelleo.local`. Upload MP3 or M4A, tap a figurine on the reader, bind it to a list.

Library lives in `/library.json` on the SD card. Back it up by copying the card.

Wi-Fi reset: hold button 3 s at power on.

---

## 5. Volume cap, two layers

| Layer | Where | Setting |
|---|---|---|
| Hardware | MAX98357A GAIN pin | Floating = 9 dB (author's choice). 100 k to VDD = 3 dB, GAIN tied to VDD = 6 dB. Pick 6 dB for under 3 and go lower if still loud. Verify against the MAX98357A datasheet table before soldering |
| Firmware | `VOLUME_MAX` in main.cpp | Library max is 21. Set 12 to 14 and measure with a phone sound meter app at 30 cm. Target under 75 dBA at the loudest knob position |

Physical knob is still exposed, so the firmware cap is the real limit. Hardware gain is the backstop if firmware ever gets replaced.

---

## 6. Toddler changes to the stock design

| Area | Change |
|---|---|
| Lift to pause | Enable `poseRequise` in the web page settings. Pause on lift, resume on place. Already in firmware |
| Stickers | Embedded or sealed, never exposed (see flags) |
| Figurines | Wooden peg dolls or Schleich style animals 6 cm plus. Nothing smaller than a 35 mm film canister |
| Cards | Credit card size, laminated, rounded corners |
| Knob | Cap the pot with a large knob. Switch at the bottom of travel is the off switch, so the child can turn it off. Acceptable, it is a safe state |
| Button | Large arcade button, single action. Short press pause, long press next |
| Case | No screws reachable from outside, speaker grille holes under 5 mm, USB-C port recessed |
| Mass | Add weight to the base so it is not thrown easily |

---

## 7. Enclosure options (no printer)

| Option | Cost | Effort | Notes |
|---|---|---|---|
| **Print service** using author's STLs (`boitier/marelleo-corps.stl`, `-base.stl`, `-molette.stl`) | 60 to 120 AUD | Low | 120 x 120 x 80 body plus 29 mm base, PLA, 3 walls, 15 % infill. Local Perth print services or a library makerspace |
| Wooden box from a craft store, drill and route | 20 to 40 AUD | Medium | NFC reads fine through 3 to 6 mm ply. Toddler proof if glued and sanded |
| Buy a Bambu A1 Mini | ~450 AUD | High upfront | Author printed on an A1. Pays off only if you will print other things |

Reader placement: PN532 antenna flat against the underside of the top face, under an engraved or painted landing circle. 3 mm PLA or 4 mm ply is fine.

---

## 8. Suggested extras you did not list

| Extra | Effort | Worth it |
|---|---|---|
| Battery gauge | Medium. Proto has none. Add a 100 k / 100 k divider from cell + to GPIO 1 and set `PIN_BATTERIE = 1` in main.cpp | Yes, otherwise you find out when it dies |
| Low battery chime | Low once gauge exists. Firmware already has a chime module | Yes |
| Default figurine | Built in. Any unknown tag plays a default list | Yes, zero config for new cards |
| Shuffle per figurine | Built in (`aleatoire`) | Nice for music cards |
| Loop prevention | Built in. A list ends and stops, no repeat. Author's bedtime logic | Keep |
| Quiet hours | Not in firmware. Would need an RTC or NTP and a change in main.cpp | Later, if at all |
| Headphone jack | No. Not for under 3 |

---

## 9. Phased build

| Phase | Done when | Est time |
|---|---|---|
| 0 Order parts | Everything in section 2 on the bench | 1 to 2 weeks shipping |
| 1 Bring-up on breadboard | Serial shows SD mounted, PN532 firmware version, a test MP3 plays | 1 evening |
| 2 Library and web page | Two figurines and two cards bound, lift to pause working, volume cap measured | 1 evening |
| 3 Power | Runs 4 hours plus on battery, charges via USB-C, pot switch kills it | 1 evening |
| 4 Perfboard | Breadboard replaced, everything tied down | 1 evening |
| 5 Enclosure | Sealed case, reader under the circle, no exposed parts | 1 weekend |
| 6 Hardening | OTA disabled, box on IoT VLAN, SD backup taken, tags sealed | 1 hour |

---

## 10. Repo layout for the new GitHub repo

```
storybox/
  README.md            this plan, trimmed
  docs/BOM.md
  docs/WIRING.md
  docs/BUILD-LOG.md    dated notes per phase
  firmware/            git submodule or fork of Zebra-Rodeo/marelleo
  patches/             version.h and main.cpp diffs (OTA off, VOLUME_MAX)
  enclosure/           STLs or ply cutting plan
  content/             nothing copyrighted, just a folder structure example
```

## 11. Open questions for you

1. Fork the Marelleo repo into your GitHub, or submodule it? Fork is simpler for the two line patch.
2. Enclosure: QIDI Plus5, decided.
3. Cell size: 3000 mAh, decided.

---

## 12. Next steps (as of 1 Nov 2026)

| # | Step | Owner | Blocked by |
|---|---|---|---|
| 1 | Create GitHub repo `storybox`, add to this session, fork Zebra-Rodeo/marelleo | You | Nothing |
| 2 | Order QIDI Plus5, PLA (EN 71-3), M3 inserts and screws, surge board | You | Nothing |
| 3 | Order electronics in section 2, 3000 mAh protected cell | You | Nothing |
| 4 | Install VS Code and PlatformIO, clone fork, build `boite` env to confirm toolchain | You | Nothing, can do while parts ship |
| 5 | Apply patches: blank `MAJ_SOURCE_DEFAUT`, set `VOLUME_MAX`, enable battery gauge pins | Me, once repo exists | Step 1 |
| 6 | Print knob first as a test piece, then base, then body | You | Step 2 delivered |
| 7 | Breadboard bring-up, phase 1 | You | Step 3 delivered |
| 8 | Library, figurines, lift to pause, volume measured | You | Step 7 |
| 9 | Battery and switch, 4 hour soak | You | Step 8 |
| 10 | Perfboard, fit to case, seal tags | You | Steps 6 and 9 |
| 11 | Hardening checklist, SD backup, IoT SSID if Wi-Fi used | You | Step 10 |
