# kicad

Custom PCB design (KiCad) for the air quality monitor. Planned.

Target layout: thermal zoning so the SCD40 sits away from the ESP32 and
regulator heat and the SPS30 exhaust, on a slot-isolated peninsula, with the
SPS30 airflow ducted to separate case vents. See the project notes for details.

## v1 design (draft, untested)

**Untested.** Nothing below has been built yet. Parts of it depend on the
breadboard tests in the main README's "To test" section, and change if a test
says so.

One PCB for every build, ordered in a batch of 5 and hand-soldered (through-hole
where possible, no assembly service). Each build gets its own 3D-printed case:

| Build | SCD40 | OLED | SPS30 |
|---|---|---|---|
| Living room | yes | yes | yes |
| Bedroom | yes | no | no |
| Portable (power bank, car) | yes | yes | no |

Home Assistant is the control plane, so there are no buttons or buzzers. Unused
dev board pins keep their holes for wiring something on later. Battery power is
for a later version.

### Power

- **Input:** a power-only USB-C socket (6 pins, for example the GCT USB4125
  footprint in KiCad's `Connector_USB` library), 5V, no USB PD. CC1 and CC2
  each get their own 5.1 kΩ resistor to GND, so a USB-C charger turns on 5V.
- **5V rail** feeds the SPS30 and both 3.3V regulators directly.
- **Dev board 5V pin** (J1 pin 14) is fed from the 5V rail through a Schottky
  diode, which a jumper cap bypasses by default (see below). With the cap
  pulled, plugging in the dev board's own USB at the same time can't push
  current back into the PCB's USB-C. In Espressif's DevKitC-1 schematic v1.4,
  each USB port reaches `VCC_5V` through a 1N5819, and `VCC_5V` is the 5V pin
  with no diode in between.
- **Two AP2112K-3.3** (SOT-23-5) from the 5V rail, each with EN tied to VIN and
  1 µF on input and output, instead of the dev board's 3V3, which also feeds the
  WiFi:
  - **3V3_SCD** feeds only the SCD40. Pending the LDO test.
  - **3V3_AUX** feeds the OLED and the Qwiic sockets.
- **Fallback to the dev board's rails:**
  - each 3.3V rail can be switched to the dev board's 3V3 pin with a 3-pad
    solder jumper (`SolderJumper_3_Bridged12`): a thin trace joins the LDO side
    from the factory, so the board works without soldering a jumper; to switch,
    cut it and put a solder blob on the dev board side. Solder jumpers,
    not pin headers with caps, because a push-fit contact in the SCD40's supply
    is the kind of fault behind the earlier stalls;
  - a 2-pin header with a jumper cap (`Jumper_2_Bridged`) across the Schottky
    diode, cap on by default: the dev board gets the full 5V, not 5V minus the
    diode drop, likely too little for its AMS1117 (an ESP32-S3 board went into
    a brownout loop at 4.9V on its 5V pin). A loose cap only puts the diode back
    in the path. Pull the cap before plugging in both USB cables, otherwise the
    device on the PCB's USB-C gets fed backwards. With the cap on, the dev board's
    own USB also powers the whole PCB;
  - two optional electrolytics (C6 100µF, C7 470µF, not fitted) on the dev
    board's 3V3 pin: that pair (570µF) stopped the same S3 board's brownout loop
    at 5.0V.

### Dev board

ESP32-C6-DevKitC-1 or a pin-compatible copy, soldered through its pin headers:
2 rows of 16 pins at 2.54 mm, rows 22.86 mm apart, board 25.4 × 51.8 mm
(Espressif dimension drawing v1.2). Pin 16 on both rows stays unconnected:
it is NC on Espressif's board and GND on the QSZNTEC copy.

Every header hole the PCB connects to (5V, 3V3, SDA, SCL, and GND on two pins
with one jumper each) goes through a cuttable solder jumper
(`SolderJumper_2_Bridged` in KiCad's `Jumper` library), so another board's pin
in that position is never shorted to anything. A board with a
different pinout fits by cutting the jumper and wiring its pin to the jumper's
pad. This does not fix a different row distance.

I2C stays on the pins the firmware uses today: SDA GPIO21, SCL GPIO22, at
100 kHz (the SPS30 maximum).

### Sensors and display

- **SCD40:** the breakout (pins SDA, SCL, VDD, GND; it already has its own
  pull-ups R1/R2 and C1), soldered on permanently through its pin header. Far
  from the dev board, both AP2112Ks and the SPS30 exhaust.
- **OLED:** 4 holes at the board edge in the module's order GND, VDD, SCK, SDA.
  Solder the module straight in, run wires to place it elsewhere in a case, or
  leave it out. No copper or parts under the module.
- **SPS30 (optional):** JST ZH 5-pin socket (B5B-ZR) for its ZHR-5 cable. Pins
  per the SPS30 datasheet v2.0: 1 VDD (5V, 4.5 to 5.5V), 2 SDA, 3 SCL, 4 SEL,
  5 GND. SEL goes to GND to select I2C. Up to 80 mA during fan start. Sensirion
  recommends UART over I2C for cables longer than 20 cm.
- **Qwiic sockets (1 or 2):** for trying out other I2C sensors (the ENS160 and
  AHT21 module, for example). JST SH 4-pin, 1 mm pitch (SM04B-SRSS-TB), pin
  order GND, 3.3V, SDA, SCL per SparkFun. These are small SMD parts and the
  hardest joints on the board.

### Still open

- Pull-ups: the SCD40 breakout and probably the OLED module bring their own,
  and the SPS30 needs some on the bus. Whether the PCB adds pull-ups is
  decided by the bus test.
- Exact USB-C socket, Schottky diode and capacitor parts, picked with the
  footprints.
- ESP32-S3 dev board (2 × 22 pins, rows roughly 25.4 mm apart from a photo,
  not measured) is not supported in v1.
