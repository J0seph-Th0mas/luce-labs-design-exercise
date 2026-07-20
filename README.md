wher# Luce Labs Design Exercise — Rack Sensing & Control Slice

**Slice:** substrate moisture + air temperature → irrigation valve, correlation logic in Home Assistant.

## Repo structure

```
/docs
  component-matrix.md     — Deliverable 1: component selection matrix, picks, weakest-buy analysis
  design.md                — Deliverable 2: block diagram, ESPHome bridge design, correlation logic,
                              failure modes, bench-test order, time accounting
  ai-usage-appendix.md     — Deliverable 4: AI tools/prompts used, an error caught, what wasn't delegated
/ha-config
  helpers.yaml              — input_boolean/input_number helpers for bench simulation
  template_sensors.yaml     — "effective" sensors (real-vs-simulated) + sensor health check
  automations.yaml          — Deliverable 3: correlation logic, hysteresis, failsafes, alerts
  dashboard.yaml            — Lovelace dashboard sketch
README.md                   — this file
```

## Architecture summary

One rack slice, all decision logic living in Home Assistant:

- **DFRobot SEN0308** capacitive soil moisture probe → **ESP32 + ESPHome** bridge → exposed to HA as
  native entities over the ESPHome API (the required non-native/bridged sensor)
- **SONOFF SNZB-02P** Zigbee temp/humidity sensor → HA via ZHA/Zigbee2MQTT (native)
- **Shelly 1 Gen3** relay + 24VAC solenoid valve → HA native integration (actuator)

HA combines moisture and temperature into one irrigation decision (not two independent loops), with
hysteresis and anti-short-cycling, plus independent failsafes for stuck valves, sensor faults, HA
restarts, and network loss. See `/docs/design.md` for the full reasoning and `/ha-config/automations.yaml`
for the implementation.

## Time-box accounting

See the "Time accounting" section at the bottom of `/docs/design.md`.

## AI disclosure

AI tools (Claude) were used throughout for sourcing research, HA/ESPHome config drafting, and
self-review. Full disclosure, prompts, and a caught error are in `/docs/ai-usage-appendix.md`.
