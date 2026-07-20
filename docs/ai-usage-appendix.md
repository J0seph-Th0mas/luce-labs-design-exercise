# AI-Usage Appendix

## Tools used
**Claude (Anthropic, web chat)** — used throughout, for three categories of work:
1. **Sourcing research** — finding real, current, purchasable components (DFRobot, Adafruit, SONOFF, Aqara, Shelly) with live web search rather than relying on memorized/potentially outdated product info, since specs and prices change and a stale or invented SKU would fail the exercise's own "don't hallucinate a product" test.
2. **HA/ESPHome config drafting** — first-pass YAML for the ESPHome bridge, template sensors, automations, and dashboard, which I then reviewed, corrected, and adapted to the specific correlation logic (moisture + temperature thresholds, hysteresis band, anti-short-cycling window).
3. **Explanation/self-check** — asked Claude to walk back through its own generated files line-by-line so I could verify I actually understood every trigger/condition/action before treating it as mine to defend live.

## Prompts that mattered (representative, not exhaustive)
1. *"[Uploaded the brief PDF] — read this carefully, note down what needs to be done, the criteria, understand it and tell me in detail what it's about."* — used to get a structured breakdown of the deliverables and scoring weights before doing any design work, so nothing in the brief got missed.
2. *"So what are the different options I have for picking the two sensors and one actuator — give reasons why I should pick it up and why I should avoid [each]."* — used to force a comparison of multiple valid designs (temp+humidity/fan, moisture+temp/irrigation, CO2+light/dimming, pH+moisture/dosing) against the brief's actual scoring weights, rather than defaulting to the first idea.
3. *"Component selection matrix... Score them on protocol/HA integration, data quality, reliability, cost, availability. Pick one each and defend the choice. Which component is the weakest buy — and at what point would you build it instead?"* — used directly from the brief's own language to make sure the AI's output matched the deliverable's actual grading criteria, not a generic comparison table.

## A concrete case where Claude was wrong
While drafting `helpers.yaml`, Claude generated a file with **two separate top-level `input_boolean:` blocks** in the same YAML file (one for the simulation toggle, a second one added later for the manual override toggle). This is invalid — Home Assistant YAML only allows one instance of a given top-level key per file; a second `input_boolean:` block would silently overwrite or conflict with the first when HA parses it, rather than merging. I caught this by re-reading the file before treating it as final, noticed the duplicate key, and asked Claude to merge them into a single `input_boolean:` block with both helpers nested underneath — which it did correctly on the second pass. Lesson: even correctly-formatted-looking YAML from an AI needs to be actually reviewed against the target application's parsing rules, not just visually skimmed.

## What I deliberately did not delegate
- **Final component picks and their tradeoffs** — I directed which design (moisture+temp→irrigation) to pursue and pushed back on/selected among the options Claude presented; I didn't let it pick unilaterally.
- **The specific threshold numbers** (30%/45% moisture, 15°C temperature veto, 30-min cooldown, 3-min max runtime) — reviewed and will defend these live as my own judgment calls, not just accepted defaults.
- **Understanding every automation before submission** — walked through each YAML file's logic explicitly rather than treating generated config as a black box, specifically because I'll be asked to explain it live in the interview.
- **The architectural-alignment decision** (bridging via ESPHome native API vs. MQTT discovery vs. a Matter bridge) — reviewed the reasoning Claude gave and would defend or challenge it based on my own understanding of the tradeoffs, not just repeat it.
