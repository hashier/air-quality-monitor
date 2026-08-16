# air-quality-monitor

DIY air quality monitor on an ESP32-C6, running ESPHome and reporting to Home Assistant.

## Hardware

- **ESP32-C6** dev board (`esp32-c6-devkitc-1`)
- **Sensirion SCD40**: CO2, temperature, humidity (I2C `0x62`)
- **Sensirion SPS30**: particulate matter PM1.0 / 2.5 / 4.0 / 10.0 (I2C `0x69`, 5V)
- **SSD1306** 128x64 OLED (I2C `0x3C`)
- **WS2812** RGB LED (GPIO8)

All three sensors share one I2C bus at 100 kHz (the SPS30 datasheet ceiling).

## Repo layout

- `esphome/`: firmware (ESPHome config + secrets template)
- `kicad/`: custom PCB (planned)
- `case/`: 3D-printed enclosure (planned)

## Firmware

```fish
cd esphome
cp secrets.yaml.example secrets.yaml   # then fill in real values
esphome run air-quality-monitor.yaml
```

`secrets.yaml` holds WiFi / API / OTA credentials and is gitignored. Never commit it.

## Notes

- **SCD40 auto-recovery:** the sensor can get knocked out of periodic measurement (a flaky power connection resets it). A YAML watchdog detects CO2 going stale and re-inits it over raw I2C (stop, then start). Two Home Assistant diagnostics track this: `CO2 Stall Count` and `CO2 Currently Stalled`.
- **Temperature offset** (`6.5C`) is a provisional self-heating correction for the breadboard build. Re-tune in the final enclosure by comparing side by side against a reference.
- **Warm-up filters** drop the SCD40 startup transient (temperature/humidity 5 min, CO2 30 s).

## Status

Work in progress: currently on a breadboard. Custom PCB and enclosure are next.
