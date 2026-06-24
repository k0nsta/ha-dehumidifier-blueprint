# Dehumidifier humidity control + tank-full guard — Home Assistant blueprint

Govern any `humidifier` (dehumidifier) from a **room humidity sensor** with proper
**hysteresis** (run above a high threshold, stop at a lower one), optionally **force a run
mode** so the unit's own humidistat can't stop it early, and **stop + alert** when the water
tank is full. A periodic safety re-check keeps a steadily-humid room from being missed.

## Install

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fk0nsta%2Fha-dehumidifier-blueprint%2Fblob%2Fmaster%2Fblueprints%2Fdehumidifier_humidity.yaml)

Or **Settings → Automations & Scenes → Blueprints → Import Blueprint** and paste:

```
https://github.com/k0nsta/ha-dehumidifier-blueprint/blob/master/blueprints/dehumidifier_humidity.yaml
```

## Prerequisites

- A dehumidifier exposed to HA as a **`humidifier`** entity (e.g. a Tuya unit via
  [LocalTuya](https://github.com/xZetsubou/hass-localtuya) — local control, no cloud).
- A **humidity `sensor`**. The unit's onboard sensor works, but a separate sensor placed away
  from the dry-air outlet tracks the *room* far better.
- *(Optional)* a **`binary_sensor`** that reports tank-full (often a Tuya fault DP).

## How it works

```
RH rises above HIGH (held N min) ──> turn on  (+ force mode, + on-actions)
RH falls to/below LOW (held N min) ─> turn off (+ off-actions)
   the HIGH↔LOW gap = hysteresis  → no short-cycling

tank sensor → full (30 s) ──> turn off, run tank-full actions (once)
tank sensor → cleared    ──> re-evaluate humidity, resume if still humid

every N min (optional) ────> re-evaluate, so steady-high RH still triggers
```

- **HA governs, not the unit.** The unit's humidistat usually stops it relative to the *dry
  outlet air*. Set **Force run mode** to the unit's continuous mode (e.g. `Continuous`, or the
  Tuya typo `Continuities`) so it keeps running until HA's LOW threshold off the *room* sensor.
- **Tank full takes priority** — the unit is stopped and the alert fires once on the
  transition; the unit won't restart while the tank reads full, even if humidity is high.
- **`mode: queued`** so a tank-full event and a humidity change don't clobber each other.

## Inputs

| Section | Key inputs |
|---|---|
| 1 · Dehumidifier | `humidifier` entity, optional **force run mode** (text) |
| 2 · Humidity control | Humidity `sensor`, ON-above %, OFF-at % (hysteresis gap), sustain-for, optional on/off actions |
| 3 · Periodic re-check | Enable toggle, interval (10/15/30/60 min) |
| 4 · Water tank | Optional tank `binary_sensor`, state meaning "full", tank-full actions |

## Notes & caveats

- **Keep OFF a few % below ON.** That gap is the anti-short-cycle hysteresis; without it the
  unit chatters around a single setpoint.
- **Force mode is unit-specific.** It must exactly match a mode the unit reports (case-sensitive).
  Leave it empty to never call `set_mode`.
- **No tank sensor?** Leave it empty — the tank branch is skipped entirely.
- **Run one instance per dehumidifier.** Each gets its own sensor and thresholds.
- The periodic re-check is idempotent (it only acts when state actually needs to change), so a
  shorter interval just means faster recovery after restarts, not extra device chatter.

## License

MIT — see [LICENSE](LICENSE).
