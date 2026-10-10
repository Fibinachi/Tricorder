# Tricorder Development Plan

**Target:** Dragon Con 2027 (Labor Day weekend, September 2027)
**Concept:** Working tricorder in a hollowed-out Playmates toy shell. Tom Paris cosplay.

---

## PCB Functional Design

### Board Overview
- **Form factor:** Custom shape to match Playmates tricorder toy's existing PCB mounting points
- **Layers:** 4-layer (signal, ground, power, signal) — needed for LoRa RF integrity
- **MCU:** ESP32-S3 (native USB, BLE 5.0, sufficient GPIO, dual-core for sensor + radio tasks)

### Functional Blocks

**1. Power**
- LiPo pouch cell (2000mAh, flat) with protection circuit
- USB-C charging (MCP73831 or similar, 500mA charge rate)
- 3.3V LDO for logic, separate 3.3V switching regulator for FLIR/sensors (noise isolation)
- Power domains: always-on (MCU deep sleep), sensor bus (switched via MOSFET), LoRa (switched), display (switched)
- Battery fuel gauge (MAX17048) for accurate % on OLED

**2. Processing & Comms**
- ESP32-S3: main MCU, BLE to phone, sensor polling, OLED driving
- SX1262 LoRa transceiver (915 MHz US) for Meshtastic
- PCB trace antenna for LoRa (or u.FL for external whip — better range at Dragon Con)
- BLE antenna: ESP32-S3 PCB antenna is fine

**3. Sensor Bus (I2C, primary)**
| Sensor | Part | Function |
|---|---|---|
| Baro/temp/humidity | BME280 | Pressure, altitude, temp, humidity |
| Air quality | SGP30 | VOCs, eCO2 |
| Particulates | PMS5003 | PM1.0/2.5/10 (UART, not I2C) |
| Light/UV | VEML6075 | UV index, ambient light |
| Magnetometer | LIS3MDL | Compass, EMF detection |
| ToF laser | VL53L0X | Distance measurement |

**4. Sensor Bus (SPI)**
| Sensor | Part | Function |
|---|---|---|
| Thermal camera | FLIR Lepton 3.5 | Thermal imaging (or MLX90640 for lower cost) |
| Display | SSD1351 OLED | Color readout, 128x128 |

**5. Other Peripherals**
- GPS: NEO-M8N (UART) — position, time sync
- Microphone: SPH0645 I2S MEMS — sound level, frequency analysis
- NFC: PN532 (I2C/SPI) — tag scanning
- Red laser diode (GPIO + transistor driver) — visible targeting dot
- Buzzer/piezo (GPIO + PWM) — tricorder chirp on scan
- RAZR keypad interface (GPIO matrix) — if using RAZR shell instead

**6. User Interface**
- OLED: live sensor readouts, Meshtastic messages, scan results
- Keypad/buttons: mode select, scan trigger
- Phone app (BLE): full LCARS dashboard, data logging, Meshtastic chat

### Firmware Architecture (ESP32-S3)
- **Core 0:** Sensor polling, OLED updates, power management
- **Core 1:** Meshtastic stack, BLE GATT server
- **Modes:** Scan (all sensors active), Idle (deep sleep, LoRa RX only), Mesh relay

---

## Development Phases

### Phase 1: Prototype (Now – Dec 2026)
- [ ] Breadboard ESP32-S3 + key sensors (BME280, VL53L0X, OLED)
- [ ] Validate BLE phone connection, basic data flow
- [ ] Test Meshtastic firmware on ESP32-S3 + SX1262 dev board
- [ ] Measure toy shell interior, confirm PCB dimensions and mount points
- [ ] Order FLIR Lepton or MLX90640 (long lead time)

### Phase 2: PCB Design (Jan – Feb 2027)
- [ ] KiCad schematic: all functional blocks
- [ ] PCB layout: match toy mounting points, RF keepouts for LoRa
- [ ] Design review: power integrity, antenna placement, thermal
- [ ] Order from JLCPCB or PCBWay (4-layer, lead-free)
- [ ] Order stencil for SMD assembly

### Phase 3: Bring-Up (Mar – Apr 2027)
- [ ] Assemble first article (hotplate or hand solder)
- [ ] Power-on test, firmware flash, validate each sensor
- [ ] Fit check in toy shell, adjust mounting if needed
- [ ] Battery life testing, optimize sleep modes
- [ ] Spin v2 if needed (budget for one respin)

### Phase 4: Phone App (May – Jun 2027)
- [ ] BLE GATT service definition (sensor characteristics)
- [ ] LCARS-style dashboard UI (Flutter or native)
- [ ] Data logging + export (CSV)
- [ ] Meshtastic message interface
- [ ] Sound effects (tricorder chirp, button blips)

### Phase 5: Integration & Polish (Jul – Aug 2027)
- [ ] Full assembly in shell, cable routing, final fit
- [ ] Field testing: range, battery, sensor accuracy
- [ ] Meshtastic mesh test with multiple nodes
- [ ] Costume: Tom Paris outfit, holster/strap
- [ ] Spare battery, charging setup for con

### Phase 6: Dragon Con (Sep 2027)
- [ ] Scan everything
- [ ] Mesh with other Meshtastic users
- [ ] Win the hallway

---

## Communicator Badge (Companion Build)

**Concept:** Toy Starfleet communicator badge, hollowed out. Tap it (chirp!) to talk over the Meshtastic mesh network. The full away-team kit.

- [ ] Source: toy communicator badge (Playmates or similar)
- [ ] Hollow out, keep the tap button
- [ ] MEMS microphone (SPH0645 or similar, I2S)
- [ ] ESP32 or nRF52 for BLE (low power, badge-sized)
- [ ] Tap button = push-to-talk: chirp sound, mic powers on
- [ ] BLE to tricorder or phone
- [ ] Phone does voice-to-text, sends text over Meshtastic LoRa mesh
- [ ] Incoming mesh messages: phone notification + optional micro speaker in badge
- [ ] Battery: small LiPo (300-500mAh), USB-C charging
- [ ] Magnetic pin back (wearable)

**Signal flow:** Tap badge → chirp → speak → BLE audio to phone → voice-to-text → Meshtastic → LoRa mesh → other badges/tricorders

### Meshtastic Ticker Display (Tricorder OLED)

**Concept:** The tricorder OLED doubles as a Meshtastic terminal. Incoming mesh messages scroll as a news ticker. Tap the communicator to reply.

- [ ] OLED layout: top section = sensor readouts, bottom strip = scrolling message ticker
- [ ] Incoming Meshtastic messages push to ticker queue (sender + text scroll across)
- [ ] Ticker pauses on new message, resumes after timeout
- [ ] Tap communicator badge → chirp → speak reply → voice-to-text → sent over mesh
- [ ] Sent confirmation blips on tricorder display
- [ ] Message history: scroll back with keypad (last 10 messages)
- [ ] Channel indicator: which mesh channel is active
- [ ] Node count: how many mesh nodes in range, displayed on ticker idle screen

---

## Budget Estimate

| Item | Est. Cost |
|---|---|
| PCB (10 pcs, 4-layer) + stencil | $80 |
| Components (sensors, MCU, LoRa, passives) | $150-200 |
| FLIR Lepton 3.5 (or MLX90640 at $60) | $60-200 |
| OLED display | $15 |
| LiPo + charger IC | $20 |
| Toy tricorder shell | $20-40 |
| GPS module | $15 |
| Miscellaneous (connectors, wire, etc.) | $30 |
| **Total** | **$400-600** |

---

## Risks

1. **FLIR lead time** — order early, have MLX90640 as backup
2. **RF interference** — LoRa + BLE + sensors on one board needs careful layout; keep analog/digital grounds separated
3. **Fit** — toy shell interior is irregular; 3D scan or careful measurement before PCB layout
4. **One respin** — budget time and money for v2; first spin rarely perfect
5. **Meshtastic + custom sensors** — Meshtastic firmware is built for specific boards; may need custom build or run sensors as separate task alongside
