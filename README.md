# Home Assistant blueprints

Three automation blueprints. Each one carries a `source_url`, so **Re-import
blueprint** picks up changes from here and keeps your inputs.

| | |
|---|---|
| [IKEA dual button → lights](#ikea-dual-button--lights) | a two-button remote driving a light group |
| [Gradual wake-up light](#gradual-wake-up-light) | ramps a light up at a set time |
| [Gradual bedtime fade](#gradual-bedtime-fade) | fades a light out, with nudges |

## IKEA dual button → lights

A two-button IKEA remote (BILRESA and friends) driving a light or light group
over Matter. All eight button events are mapped.

| | press | double press | hold |
|---|---|---|---|
| **Top** | toggle, on at your chosen tone | next preset | brighten until released |
| **Bottom** | off | night light | dim until released |

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdanechristenson%2Fha-blueprints%2Fblob%2Fmain%2Fautomation%2Fikea_dual_button_light.yaml)

Or paste this into **Settings → Automations & scenes → Blueprints → Import blueprint**:

```
https://github.com/danechristenson/ha-blueprints/blob/main/automation/ikea_dual_button_light.yaml
```

### Set it up

1. **Make a light group first.** Settings → Devices & services → Helpers →
   Create helper → Group → Light. Put the room's ceiling bulbs in it.
2. **Create automation → Use blueprint → IKEA dual button → lights**.
3. Pick the two event entities and the group. Everything else has a default.

Adding a bulb later is a change to the group, not to the automation. A bedside
lamp added to the same room is not swept up by the ceiling button, which is why
this takes a group rather than an area.

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

## Gradual wake-up light

Brings a light up from almost nothing to full at a set time, on the days you
choose, and stays off on days when an event is running on a calendar you
nominate — a holiday calendar, for instance.

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdanechristenson%2Fha-blueprints%2Fblob%2Fmain%2Fautomation%2Fgradual_wake_up_light.yaml)

Or paste this into **Settings → Automations & scenes → Blueprints → Import blueprint**:

```
https://github.com/danechristenson/ha-blueprints/blob/main/automation/gradual_wake_up_light.yaml
```

### Options

Only the light is required.

| | default | |
|---|---|---|
| `light_entity` | — | the light to bring up |
| `wake_time` | 07:00 | when the ramp starts |
| `weekdays` | Mon–Fri | days it runs |
| `skip_calendar` | none | an active event here cancels the day; leave empty to disable |
| `duration` | 300 s | how long the ramp takes |
| `max_brightness` | 100% | where it finishes |
| `color_temp` | 4778 K | tone held throughout |
| `reset_first` | on | jump to minimum brightness before ramping |

### Why it behaves the way it does

**The ramp is one service call, not a loop.** `light.turn_on` with a
`transition` hands the fade to the bulb, so there is no repeat running for five
minutes, nothing to drift, and a restart mid-ramp cannot leave the light
stranded at an arbitrary level.

**`reset_first` exists because a transition from full brightness does
nothing visible.** If the light is already on when the alarm comes round, the
ramp has nowhere to go; dropping to brightness 1 first makes the sunrise work
regardless of how the light was left.

**The calendar check is a condition, not a trigger.** It is read at wake time
only, so creating a holiday event the night before is enough — nothing has to
re-evaluate at midnight.

**An empty calendar is handled explicitly.** The template passes when
`skip_calendar` is blank, so the input stays genuinely optional rather than
needing a dummy entity.

## Gradual bedtime fade

Takes a light from full to off over ten minutes, pulsing as it goes so the fade
is noticed rather than just happening. Does nothing if the light is already
off, and switching the light off by hand abandons the fade.

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdanechristenson%2Fha-blueprints%2Fblob%2Fmain%2Fautomation%2Fgradual_bedtime_fade.yaml)

Or paste this into **Settings → Automations & scenes → Blueprints → Import blueprint**:

```
https://github.com/danechristenson/ha-blueprints/blob/main/automation/gradual_bedtime_fade.yaml
```

### The ten minutes

| | |
|---|---|
| 0:00 | jumps to 100% |
| 0:00 → 5:00 | fades to 50% |
| 5:00 → 9:00 | each minute: pulses, then drops 5% — one pulse in the first minute, four in the fourth |
| 9:00 → 9:50 | fades to 10% |
| 9:50 | one last pulse |
| 9:51 → 10:00 | down to 1%, then off |

### Options

Only the light is required.

| | default | |
|---|---|---|
| `light_entity` | — | one light or several |
| `bedtime` | 22:00 | when the fade starts |
| `weekdays` | every day | days it runs |
| `pulse_boost` | 15% | how far a pulse jumps above the current level |

### Why it behaves the way it does

**Pulses get more frequent as time runs out** — one in the first minute, four
in the fourth. A steady fade is easy to miss; an accelerating one reads as
urgency without needing sound.

**Switching the light off by hand ends it.** The light's own `off` is a
trigger, and `mode: restart` means that trigger cancels the fade in progress.
The new run then stops at its first condition, which requires the bedtime
trigger. No helper, no flag.

**It checks the light is on before starting.** A bedtime fade on an already-dark
room would switch the light *on* at 100%, which is the opposite of the point.

**Timings are fixed rather than scaled.** The pulse schedule is tuned to the
ten minutes; making the duration an input would mean either stretching the
pulses until they stop reading as urgent, or recomputing the whole ladder.
Change `transition` and `delay` in step if you want a different length.

**With several lights, all of them must be on for the fade to start.** A state
condition over a list requires every entity to match. That suits a set of bulbs
in one room, which is the intended use.

## Licence

MIT — see [LICENSE](LICENSE). Copy it, change the presets, ship it in your own
config; attribution is the only condition.
