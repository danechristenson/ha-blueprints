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
| **Top** | toggle, on at your chosen tone | next preset | brighten until released |
| **Bottom** | off | night light | dim until released |

All eight button events are used.

Holding ramps continuously and stops on release. That needs `mode: restart` —
the release event re-enters the automation, abandoning the running loop. The
loop is also step-bounded, so a lost release event stops the ramp rather than
running it to the limit. Dimming stops at the dimmest the bulb will hold, so
holding too long never loses the light.

Bottom single press is a plain **off** rather than a dim step, since holding
already dims. In the dark that gives one button that always turns the light
off, with no risk of toggling it back on.

The turn-on tone defaults to 2000 K, below most bulbs' floor. Home Assistant
clamps it, so it lands on the warmest each bulb can manage — set it exactly if
you want a specific tone.

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
