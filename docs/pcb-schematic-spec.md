# Tricorder PCB Schematic Specification

**Board:** Tricorder Main (ESP32-S3)
**Form factor:** Custom, matches Playmates toy tricorder PCB mounting points
**Layers:** 4 (Top signal, GND plane, 3V3 power plane, Bottom signal)
**Target:** JLCPCB or PCBWay, lead-free, 1.6mm

---

## 1. Bill of Materials

### MCU & Radio
| Ref | Part | Package | Notes |
|---|---|---|---|
| U1 | ESP32-S3-WROOM-1-N8R2 | Module | 8MB flash, 2MB PSRAM, PCB antenna |
| U2 | SX1262 | QFN-24 | LoRa transceiver, 915MHz |
| ANT1 | 915MHz PCB trace antenna | — | Or u.FL for external whip (preferred for range) |

### Power
| Ref | Part | Package | Notes |
|---|---|---|---|
| U3 | MCP73831 | SOT-23-5 | LiPo charger, 500mA, USB-C input |
| U4 | AP2112K-3.3 | SOT-23-5 | 3.3V LDO, 600mA, logic rail |
| U5 | TPS62162 | QFN-16 | 3.3V buck, 1A, sensor/FLIR rail (low noise) |
| U6 | MAX17048 | TDFN-8 | Battery fuel gauge (I2C) |
| Q1 | AO3400 | SOT-23 | Sensor bus power switch (MOSFET) |
| Q2 | AO3400 | SOT-23 | LoRa power switch |
| J1 | USB-C 16p | — | Charging + programming |
| BT1 | LiPo 2000mAh pouch | — | With protection circuit, JST-PH connector |

### Sensors (I2C Bus)
| Ref | Part | Package | Notes |
|---|---|---|---|
| U7 | BME280 | LGA-8 | Pressure/temp/humidity |
| U8 | SGP30 | DFN-6 | VOCs, eCO2 |
| U9 | VEML6075 | OPLGA-10 | UV index, ambient light |
| U10 | LIS3MDL | LGA-16 | Magnetometer |
| U11 | VL53L0X | LGA-12 | ToF laser rangefinder |

### Sensors (SPI Bus)
| Ref | Part | Package | Notes |
|---|---|---|---|
| U12 | FLIR Lepton 3.5 | Socket | Thermal IR camera (or MLX90640) |
| U13 | SSD1351 OLED 128x128 | Module | Color display |
| U18 | OV5640 NoIR | Module | Near-IR night vision camera (no IR filter) |
| U19 | OV5640 standard | Module | Visible + UV fluorescence camera |

### Visual Scanner Array
| Mode | Hardware |
|---|---|
| Thermal IR | FLIR Lepton 3.5 |
| Night vision | NoIR camera + 850nm IR LED illuminators (D2, D3) |
| UV fluorescence | Standard camera + 365nm UV LED (D4) |
| Visible | Standard camera + white LED (D5) |

### Sensors (UART)
| Ref | Part | Package | Notes |
|---|---|---|---|
| U14 | NEO-M8N GPS | Module | With patch antenna or u.FL |
| U15 | PMS5003 | Module | Particulate matter (external, cabled) |

### Other Peripherals
| Ref | Part | Package | Notes |
|---|---|---|---|
| U16 | SPH0645LM4H | — | I2S MEMS microphone |
| U17 | PN532 | QFN-40 | NFC/RFID reader |
| J2 | MicroSD push-push | — | SPI data logging |
| D1 | Red laser diode 650nm 5mW | — | Targeting dot (Q3 driver) |
| Q3 | 2N2222 | SOT-23 | Laser diode driver |
| BZ1 | Piezo buzzer | — | Tricorder chirp (PWM) |
| J3 | JST-PH 6p | — | RAZR keypad matrix input |

---

## 2. Pin Assignments (ESP32-S3)

### I2C Bus 0 (Sensors)
| Signal | GPIO | Notes |
|---|---|---|
| SDA | GPIO8 | 4.7k pull-ups |
| SCL | GPIO9 | 4.7k pull-ups |

Devices on bus: BME280 (0x76), SGP30 (0x58), VEML6075 (0x10), LIS3MDL (0x1C), VL53L0X (0x29), MAX17048 (0x36)

### SPI Bus 2 - FSPI (FLIR + OLED + MicroSD, shared with individual CS)
| Signal | GPIO | Notes |
|---|---|---|
| SCK | GPIO12 | |
| MISO | GPIO13 | |
| MOSI | GPIO11 | |
| CS_FLIR | GPIO10 | |
| CS_OLED | GPIO14 | |
| CS_SD | GPIO21 | |
| DC_OLED | GPIO7 | Data/command |
| RST_OLED | GPIO6 | Reset |

### SPI Bus 3 - HSPI (LoRa SX1262, dedicated)
| Signal | GPIO | Notes |
|---|---|---|
| SCK | GPIO36 | |
| MISO | GPIO37 | |
| MOSI | GPIO35 | |
| CS_LORA | GPIO38 | |
| DIO1 | GPIO39 | Interrupt |
| BUSY | GPIO40 | Status |
| RST_LORA | GPIO41 | Reset |
| TXEN | GPIO42 | TX/RX switch (if using) |

### UART
| Port | TX | RX | Device |
|---|---|---|---|
| UART1 | GPIO17 | GPIO18 | NEO-M8N GPS |
| UART2 | GPIO15 | GPIO16 | PMS5003 particulate |

### I2S (Microphone)
| Signal | GPIO | Notes |
|---|---|---|
| BCK | GPIO4 | Bit clock |
| WS | GPIO5 | Word select |
| DIN | GPIO6 | Data in (shared with OLED RST — use GPIO3 instead) |

**Correction:** Move OLED RST to GPIO3. I2S DIN on GPIO6.

### GPIO (Discrete)
| Signal | GPIO | Notes |
|---|---|---|
| LASER_EN | GPIO1 | Laser diode driver |
| BUZZER | GPIO2 | PWM for chirp |
| SENS_PWR | GPIO43 | Sensor bus MOSFET gate |
| LORA_PWR | GPIO44 | LoRa MOSFET gate |
| SD_DETECT | GPIO5 | MicroSD card detect (shared w/ I2S WS — use GPIO0) |

**Correction:** Move SD_DETECT to GPIO0.

### USB
| Signal | GPIO | Notes |
|---|---|---|
| USB_D- | GPIO19 | Native USB |
| USB_D+ | GPIO20 | Native USB |

### Strapping (leave floating or pull as noted)
- GPIO0: boot (with button to GND for programming)
- GPIO3: floating
- GPIO45: floating (must be low at boot — add 10k to GND)
- GPIO46: floating

---

## 3. Power Architecture

```
USB-C (5V) ──→ MCP73831 ──→ LiPo (3.0-4.2V) ──┬──→ AP2112K ──→ 3V3_LOGIC (ESP32, I2C sensors)
                                                │
                                                └──→ TPS62162 ──→ 3V3_SENSOR (FLIR, GPS, PMS5003)
                                                        (switched via Q1)

Power domains:
- ALWAYS_ON: ESP32-S3 (deep sleep capable), MAX17048, RTC
- SENSOR_BUS: All I2C/SPI sensors (Q1 switched, off in deep sleep)
- LORA: SX1262 (Q2 switched, on for RX in light sleep)
- DISPLAY: OLED (separate enable, off when idle)
```

**Current budget (active scan):**

| Domain | Current |
|---|---|
| ESP32-S3 (active, BLE+WiFi off) | 80mA |
| LoRa RX | 15mA |
| LoRa TX (22dBm) | 120mA (brief) |
| FLIR Lepton | 150mA |
| GPS (acquisition) | 30mA |
| OLED | 20mA |
| I2C sensors (all) | 10mA |
| PMS5003 (fan) | 100mA |
| **Total active** | **~400mA** |
| Deep sleep | <100µA |

**Battery life:** 2000mAh / 400mA = ~5 hours continuous scan. Realistic with duty cycling: 8-10 hours.

---

## 4. Layout Priorities

1. **RF keepout:** No copper under ESP32-S3 antenna area or 915MHz trace antenna. Follow module datasheet keepout dimensions exactly.
2. **LoRa antenna:** Place at board edge, away from digital noise. u.FL + external whip preferred over trace antenna for range.
3. **Ground plane:** Solid, unbroken. No splits. Via-stitch around RF sections.
4. **Power:** Star routing from regulators. Bulk caps (10µF) at each regulator output, 100nF at every IC power pin.
5. **FLIR:** Keep away from heat sources (regulators, ESP32). Thermal isolation slot in PCB if possible.
6. **Magnetometer (LIS3MDL):** Away from inductors, high-current traces, and the LiPo. At least 20mm from any magnetic source.
7. **USB-C:** ESD protection diodes on D+/D-.
8. **Mounting holes:** Match toy shell posts exactly. Measure twice.

---

## 5. Communicator Badge (ESP32-C3)

**Board:** 2-layer, minimal, fits inside Playmates communicator toy

### BOM (Badge)
| Ref | Part | Notes |
|---|---|---|
| U1 | ESP32-C3-MINI-1 | BLE, RISC-V, tiny |
| U2 | SPH0645LM4H | I2S MEMS mic |
| U3 | MCP73831 | LiPo charger |
| SPK1 | 8Ω 0.5W micro speaker 15mm | Incoming message audio |
| BT1 | LiPo 400mAh pouch | Flat, badge-sized |
| SW1 | Existing toy button | Push-to-talk (reuse) |

### Pin Assignments (ESP32-C3)
| Signal | GPIO | Notes |
|---|---|---|
| I2S BCK | GPIO4 | Mic clock |
| I2S WS | GPIO5 | Mic word select |
| I2S DIN | GPIO6 | Mic data |
| PTT_BTN | GPIO3 | Toy button (interrupt, wake from sleep) |
| SPK_PWM | GPIO20 | Speaker driver (via transistor) |
| CHIRP | GPIO21 | Piezo for tap chirp |
| USB D-/D+ | GPIO18/19 | Programming/charging |

### Badge Power
- Deep sleep: <10µA (button wake only)
- BLE connected idle: ~15mA
- Active (mic + BLE TX): ~80mA
- 400mAh = ~5 hours talk, days on standby

---

## 6. Reference: Bluetooth Combadge Speaker

Charles noted seeing a Bluetooth speaker shaped like a combadge. This validates the form factor — a badge with a speaker is a known product category. Our badge adds: microphone, ESP32-C3, BLE to phone, Meshtastic integration. The commercial versions are just speakers; ours is a communicator.

---

## 7. Firmware Notes

### Tricorder (ESP32-S3)
- **Core 0:** Sensor polling loop (I2C/SPI/UART), OLED rendering, power management, SD logging
- **Core 1:** Meshtastic stack, BLE GATT server, LoRa TX/RX
- BLE GATT: sensor data characteristics (notify), command characteristic (write: mode select, scan trigger)
- SD logging: CSV, one file per day, GPS timestamp on each row

### Badge (ESP32-C3)
- Deep sleep until PTT button interrupt
- Wake → chirp → BLE connect to phone → stream mic audio → phone does voice-to-text → Meshtastic send → confirmation chirp → back to sleep
- Incoming: BLE notification from phone → play through speaker

---

*This spec is the design input for KiCad. It does not replace the .kicad_pcb layout file — that requires hands-on placement and routing.*
