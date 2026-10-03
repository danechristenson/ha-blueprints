# Home Assistant blueprints

## IKEA dual button → lights

A two-button IKEA remote (BILRESA and friends) driving a light or light group
over Matter. All eight button events are mapped.

| | press | double press | hold |
|---|---|---|---|
| **Top** | toggle, on at your chosen tone | next preset | brighten until released |
| **Bottom** | off | night light | dim until released |

### Install

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdanechristenson%2Fha-blueprints%2Fblob%2Fmain%2Fautomation%2Fikea_dual_button_light.yaml)

Or by hand — **Settings → Automations & scenes → Blueprints → Import
blueprint**, and paste:

```
https://github.com/danechristenson/ha-blueprints/blob/main/automation/ikea_dual_button_light.yaml
```

### Set it up

1. **Make a light group first.** Settings → Devices & services → Helpers →
   Create helper → Group → Light. Put the room's ceiling bulbs in it.
2. Settings → Automations & scenes → **Create automation → Use blueprint →
   IKEA dual button → lights**.
3. Pick the two event entities and the group. Everything else has a default.

Adding a bulb later is a change to the group, not to the automation. A bedside
lamp added to the same room is not swept up by the ceiling button, which is why
this takes a group rather than an area.

### Updating

Blueprints → the three-dot menu on this blueprint → **Re-import blueprint**.
That works because the file carries a `source_url`; your inputs are kept.

### Options

Three fields are required — the two buttons and the light. The rest have
defaults:

| | default | |
|---|---|---|
| `on_kelvin` | 2000 K | tone for the top single press and the night light |
| `cycle_mode` | colour temperature | or colour, once the room has colour bulbs |
| `color_temp_presets` | 2237, 2890, 4292, 6410 | the cycle, in order |
| `color_presets` | 4 hue/saturation pairs | used when cycling colour |
| `ramp_step` | 8% | brightness change per tick while held |
| `ramp_interval` | 300 ms | time between ticks while held |
| `night_brightness` | 1% | bottom double press |
| `off_fade` | 2 s | bottom single press |

Defaults give a ramp of roughly four seconds end to end.

### Why it behaves the way it does

**Double press cycles Philips' four light recipes** — Relax 2237 K, Read
2890 K, Concentrate 4292 K, Energize 6410 K — the same way a Hue dimmer steps
through a room's scenes. Most vendors copied those. There is no equivalent
cross-vendor standard for colour, so that list is only a sensible default.

**Cycling is stateless.** It reads the light's current setting, finds the
nearest preset and advances. No counter helper per room, and changing the light
from the app or a dashboard leaves the next press sensible.

**Holding ramps until release**, which needs `mode: restart`: the release event
re-enters the automation and abandons the running loop. Under `mode: single`
the loop would hold the run and the release would be dropped, so the light
would ramp to its limit every time. The loop is step-bounded as well as
limit-checked, so a lost release event stops it rather than running it out, and
dimming stops at the dimmest the bulb will hold so holding too long never loses
the light.

**Bottom single press is a plain off**, not a dim step, because holding already
dims. In the dark that gives one button that always turns the light off, with
no risk of toggling it back on.

**The turn-on tone defaults below most bulbs' floor.** Home Assistant clamps
it, so 2000 K lands on the warmest each bulb can manage without you having to
know the number. Set it exactly if a room wants a specific tone.

**`lights` takes a single entity, not a target**, because the preset cycle and
the ramp both read the light's current state and a target has none.

## Licence

MIT — see [LICENSE](LICENSE). Copy it, change the presets, ship it in your own
config; attribution is the only condition.
