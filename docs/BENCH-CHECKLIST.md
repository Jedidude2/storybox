# Bench checklist

Tick in order. Stop at the first failure and fix it before moving on.

## Toolchain, no hardware
- [ ] VS Code and PlatformIO installed
- [ ] `git clone --recurse-submodules` done, `firmware/` populated
- [ ] Patches applied, `git -C firmware diff --stat` shows 2 files
- [ ] `pio run -e boite` ends in SUCCESS

## Flash
- [ ] DevKit on USB, `pio run -e boite -t upload` succeeds
- [ ] `pio run -e boite -t uploadfs` succeeds
- [ ] `pio device monitor` shows boot banner, version 0.7.8
- [ ] Log shows "aucune adresse de mise a jour enregistree" or no update attempt at all

## Storage
- [ ] SD card FAT32, one test MP3 in root
- [ ] Log shows SD mounted, not "repli sur la memoire flash"

## NFC
- [ ] PN532 DIP switches on HSU
- [ ] Log shows PN532 firmware version at boot
- [ ] Tap a tag, log shows UID

## Audio
- [ ] Amp VIN on 5 V, SD on GPIO 4
- [ ] Startup chime heard
- [ ] Volume line in log changes as the knob turns, range 3 to 13

## Controls
- [ ] Short press pauses and resumes
- [ ] Long press skips track
- [ ] Knob at minimum goes to sleep after 1.5 s

## Wi-Fi and library
- [ ] AP `Marelleo-XXXX` appears on first boot
- [ ] Joined IoT SSID, `http://marelleo.local` loads
- [ ] Upload one MP3, bind to a tag, tag plays it
- [ ] Lift to pause enabled, lift pauses, replace resumes
- [ ] `library.json` copied off the SD as backup

## Volume
- [ ] Phone sound meter at 30 cm, knob max, under 75 dBA
- [ ] If over, lower VOLUME_MAX or set gain pin to 3 dB, reflash, remeasure

## Power
- [ ] Lipo Rider wired, cell connected, runs on battery with USB out
- [ ] Pot switch click kills power
- [ ] Charges from USB-C, Lipo Rider LED shows charging
- [ ] 4 hour playback soak on battery passes

## Hardening
- [ ] Wi-Fi credentials wiped (hold button 3 s at power on) or box stays on IoT SSID only
- [ ] Tags sealed, no exposed stickers
- [ ] No reachable screws, grille holes under 5 mm
