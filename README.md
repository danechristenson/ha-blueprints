# Home Assistant blueprints

## IKEA dual button → lights

For a two-button IKEA remote (BILRESA and friends) driving a light or light
group over Matter.

Import it into Home Assistant: **Settings → Automations & scenes → Blueprints →
Import blueprint**, and paste

```
https://github.com/danechristenson/ha-blueprints/blob/main/automation/ikea_dual_button_light.yaml
```

| | press | double press | hold |
|---|---|---|---|
| **Top** | toggle, warm, full | cycle presets | brighter |
| **Bottom** | dim a step, or full if off | night light | fade off |

Double press cycles Philips' four light recipes by default — Relax 2237 K,
Read 2890 K, Concentrate 4292 K, Energize 6410 K — the same way a Hue dimmer
steps through a room's scenes. Switch `cycle_mode` to colour once the room has
colour bulbs; that list has no cross-vendor standard behind it and is just a
sensible default to edit.

Cycling is **stateless**. It reads the light's current setting, finds the
nearest preset and advances, so there is no counter helper per room and
changing the light from the app leaves the next press sensible.

`lights` takes a single entity rather than a target, because the cycle has to
read the light's current state and a target has none. Point it at a **light
group**: adding a bulb is then a change to the group, and a bedside lamp added
to the same room later is not swept up by the ceiling button.

`long_release` is deliberately unused. It only matters for hold-to-ramp, which
needs a repeat loop and a stop flag and tends to overshoot.

## Licence

MIT — see [LICENSE](LICENSE). Copy it, change the presets, ship it in your own
config; attribution is the only condition.
