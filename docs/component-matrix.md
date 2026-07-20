# Component Selection Matrix

**Rack slice:** substrate moisture + air temperature → irrigation valve, correlation logic in Home Assistant.

## Sensor 1 — Substrate Moisture (required non-native, bridged sensor)

| Product | Vendor | Price | Protocol / HA path | Data quality | 21-day reliability | Availability |
|---|---|---|---|---|---|---|
| **Gravity Waterproof Capacitive Soil Moisture Sensor (SEN0308)** | DFRobot | $18.90 | Analog → ESP32 ADC → ESPHome `sensor: - platform: adc` (custom bridge) | Capacitive, analog 0–3.3V, needs 2-point (dry/wet) calibration | IP65 waterproof, corrosion-resistant, designed for long-term outdoor gardening and automated irrigation | In stock, direct from DFRobot |
| STEMMA Soil Sensor (I2C) | Adafruit | ~$6.95 direct (often out of stock; $15–17 via resellers) | I2C → ESP32 → ESPHome via `seesaw` external component | Capacitive touch via ATSAMD10 chip, 200 (dry)–2000 (wet) range; bonus ambient temp ±2°C | No exposed metal/DC current into substrate, avoids resistive-sensor oxidation drift | Frequently out of stock |
| Gravity Analog Capacitive Soil Moisture Sensor, corrosion-resistant (SEN0193) | DFRobot | $5.90 | Analog → ESP32 ADC | Basic capacitive, no waterproof connector rating | 2–3× lifespan of resistive sensors, not IP65 | In stock |

**Pick: DFRobot SEN0308.** IP65-rated for full submersion in wet substrate over 21 days (the other two aren't). Analog output keeps the ESPHome bridge to a plain ADC read + calibration curve — no library dependency or I2C bus/address debugging inside a 6–8 hour time-box.

## Sensor 2 — Air Temperature (native)

| Product | Vendor | Price | Protocol / HA path | Data quality | Reliability | Availability |
|---|---|---|---|---|---|---|
| **SNZB-02P** | SONOFF | $15.71 | Zigbee 3.0 → any Zigbee coordinator → HA via ZHA or Zigbee2MQTT (no proprietary hub) | ±0.2°C / ±0.4°F temp, ±2% RH, Swiss-made sensor, 5s refresh | Low power, up to 4-year battery life | Widely available |
| Aqara Temperature and Humidity Sensor | Aqara | ~$20 | Zigbee 3.0 (best with Aqara hub for full feature set) | Sensirion sensor, ±0.3°C temp, ±3% RH, built-in pressure sensor | Battery-powered, similar class | Widely available |
| Aqara W100 Climate Sensor | Aqara | ~$25–30 | Matter-over-Thread native, or Zigbee via hub | ±0.2°C temp, ±2% RH, real-time LCD | Two-battery powered | Available |

**Pick: SONOFF SNZB-02P.** Best accuracy-per-dollar, and pairs directly to any Zigbee 3.0 coordinator already in HA rather than nudging toward a vendor-specific hub — matches the "swappable without redesign" architectural rule better than Aqara's hub-centric ecosystem.

## Actuator — Irrigation Valve

| Product | Vendor | Price | Protocol / HA path | Reliability | Availability |
|---|---|---|---|---|---|
| **Shelly 1 Gen3** (relay) + generic 24VAC/12VDC garden solenoid valve | Shelly + generic irrigation | $17.19 relay + ~$15–20 valve | WiFi/BT/Matter multiprotocol dry-contact relay → HA native integration | Field reports of 200+ Shelly devices in continuous home-automation use with only 2 failures in 2 years; local control persists even if HA goes offline | Widely stocked |
| Neo Smart Water Valve (NAS-WV02W) | Shelly | Premium tier | WiFi → HA native | NSF drinking-water certified, ultrasonic flow measurement, no moving parts, ±0.1 gal flow accuracy | Available, pricier |
| FrankEver 6-Zone WiFi Sprinkler Controller | Shelly-partner | Mid-tier | WiFi/MQTT → HA, no hub required | Weather-adjusted watering via historical temp/rainfall data | Available |

**Pick: Shelly 1 Gen3 relay + standard 24VAC garden solenoid valve.** A dry-contact relay is trivially fail-safe (relay off = valve closed on a normally-closed solenoid), cheap, and has field-proven reliability data. The Neo valve is nicer but overkill/pricier for a single-slice demo; the 6-zone controller is overbuilt for one valve.

## Weakest buy

**The moisture sensor (SEN0308) is the weakest buy.** Capacitive soil probes drift over weeks as they sit in wet substrate — DFRobot doesn't publish a long-term drift spec, and analog capacitive sensing is sensitive to substrate salinity/EC changes, which confounds the raw reading. Fine for a one-off 21-day pilot; not fine at fleet scale.

**Build trigger and fallback:** once this needs to run reliably across dozens of racks (past a single pilot), I'd stop buying and build a custom ESP32 + ESPHome node that reads the raw capacitive sensor but also periodically self-calibrates against a known-dry/known-wet reference cycle, logs raw ADC history to catch drift trends over time, and exposes both a raw value and a drift-corrected value as separate HA entities — so the correlation logic can treat "sensor drifting" as its own detectable failure mode instead of silently feeding degraded data into the irrigation decision.
