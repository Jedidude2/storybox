# Wiring, module build, `boite` env

| ESP32-S3 GPIO | Goes to | Note |
|---|---|---|
| 4 | MAX98357A SD | Mutes idle hiss |
| 5 | MAX98357A BCLK | |
| 6 | MAX98357A LRC | |
| 7 | MAX98357A DIN | |
| 8 | PN532 SDA/TXD | ESP RX. Crossed on purpose |
| 9 | PN532 SCL/RXD | ESP TX |
| 10 | Pot wiper | ADC1 only, GPIO 1 to 10 |
| 11 | SD CS | |
| 12 | SD SCK | |
| 13 | SD MOSI | |
| 14 | SD MISO | |
| 18 | Button, other leg to GND | Internal pull-up |
| 48 | Onboard LED | Nothing to wire |
| 3V3 | PN532 VCC, SD VCC | |
| 5V | MAX98357A VIN | From Lipo Rider 5 V out via pot switch |
| GND | Common | |

Power chain: cell -> Lipo Rider Plus BAT -> Lipo Rider 5V OUT -> pot switch -> ESP 5V pin and amp VIN. USB-C on the Lipo Rider charges the cell.

Pot: one outer pin 3V3, other outer pin GND, wiper GPIO 10.

Speaker: amp + and - to the speaker. Polarity does not matter for a single driver.

PN532 DIP switches: HSU. Positions printed on the back of the board.

MAX98357A gain pin: floating is 9 dB. GAIN to VDD is 6 dB. 100 k GAIN to VDD is 3 dB. Start at 6 dB. Verify against the MAX98357A datasheet gain table before soldering.

Free GPIO: 15, 16, 17.

Optional battery gauge: 100 k from cell + to GPIO 1, 100 k GPIO 1 to GND. Then set `PIN_BATTERIE = 1` in the non `CARTE_IMPRIMEE` branch of main.cpp. Untested on the module build.

Traps from upstream:
- RX to RX wiring: PN532 powers up and never answers.
- `nfc.begin()` runs once in setup. Power cycle after fixing wiring, a reset is not enough.
- ADC2 pins go mute when Wi-Fi starts. Pot must stay on GPIO 10.
