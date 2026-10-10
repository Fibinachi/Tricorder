# Dragon Con 2027 — Working Tricorder Cosplay

**Concept:** Dress like Tom Paris (Voyager's retro-20th-century enthusiast), carry a tricorder that actually works. Ultimate cosplay.

**Date:** Dragon Con 2027, Labor Day weekend (early September 2027)

## The Tricorder Build

**PCB decision:** Custom PCB mounted on the toy's existing PCB mounting points. Clean build, no breakout Tetris. Design in KiCad.

**Meshtastic:** Run Meshtastic on the ESP32. The tricorder becomes a LoRa mesh node — off-grid messaging between tricorders at Dragon Con. Communicator meets tricorder.

- [ ] Source: Playmates tricorder toy, hollow it out (ALTERNATE: Motorola RAZR — Charles already has one in a drawer. V3i preferred for lower antenna profile. Keypad repurposed as scan-mode buttons.)
- [ ] Display: OLED in the top half for readouts
- [ ] Core: ESP32 for sensor hub + BLE to phone
- [ ] Sensors (fit in remaining shell space):
  - [ ] Barometric pressure (BME280 — also gives temp/humidity)
  - [ ] GPS (NEO-M8N or similar)
  - [ ] FLIR thermal camera (Lepton module)
  - [ ] Air quality: PM2.5 (PMS5003), VOCs/CO2 (SGP30)
  - [ ] Pulse ox / heart rate
  - [ ] Magnetometer
  - [ ] Geiger counter module (optional, for the full effect)
  - [ ] Thermal (MLX90640 or FLIR Lepton)
  - [ ] Air quality (PM2.5, VOCs, CO2 — SGP30 + PMS5003)
  - [ ] Pulse ox / heart rate
  - [ ] Ambient temp/humidity/pressure (BME280)
  - [ ] Magnetometer (already on ESP32 or external)
  - [ ] Geiger counter module (optional, for the full effect)
- [ ] Display: small TFT in the top half for readouts
- [ ] Detachable hand scanner (TOS-style): sensor cluster on a separate BLE piece
- [ ] Phone app: unified scan dashboard, Starfleet LCARS-style UI
- [ ] Sound effects: the classic tricorder chirp on scan
- [ ] Battery: LiPo with USB-C charging, dockable

## The Costume

- [ ] Tom Paris look — Voyager uniform or his retro 20th-century outfits
- [ ] 1960s styling nod (Paris loved the era)
- [ ] Tricorder holster / shoulder strap

## Timeline

- **Now–Dec 2026:** Source parts, prototype sensor hub on breadboard
- **Jan–Mar 2027:** Enclosure, fit sensors, basic firmware
- **Apr–Jun 2027:** Phone app, LCARS UI, BLE integration
- **Jul–Aug 2027:** Polish, sound effects, field testing
- **Sep 2027:** Dragon Con. Scan everything.
