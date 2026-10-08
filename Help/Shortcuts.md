# Shortcuts

There are various shortcuts in the TDXMorph ecosystem. I have tried to keep them as simple as possible, so the only keys you will ever have to remember are <kbd>Shift</kbd>, <kbd>Ctrl</kbd> or <kbd>Alt</kbd> + <kbd>Mouse button</kbd>.

> **PROPRIETARY. Licensed, not sold.** Part of the commercial ParameterMorpher and
> SceneLauncher components, available through
> [Patreon](https://www.patreon.com/darienbrito), not from this repository.

## ParameterMorpher

### When pressing over a preset slot

* <kbd>left click</kbd> = recall the preset, morphing over its own stored time and curve
* <kbd>shift</kbd> + <kbd>left click</kbd> = store a preset, or overwrite the one in that slot
* <kbd>shift</kbd> + <kbd>right click</kbd> = delete a preset
* <kbd>shift</kbd> + <kbd>middle click</kbd> = freeze or unfreeze, which locks interaction with the elements so they cannot be nudged by accident (stored slots only)
* <kbd>ctrl</kbd> + <kbd>left click</kbd> = jump to preset (no interpolation; locked elements keep their value)
* <kbd>ctrl</kbd> + <kbd>right click</kbd> = morph over the container's global **Morph Time** instead of the preset's stored time, meant for a quick transition check
* <kbd>alt</kbd> + drag one slot onto another = swap which preset each button triggers (both slots must be stored; *Rename presets in order* bakes the new order into storage)

### When pressing over an element's name 

* <kbd>shift</kbd> + <kbd>left click</kbd> = set the element to the value and range found on creation
* <kbd>shift</kbd> + <kbd>right click</kbd> = change the element's name
* <kbd>alt</kbd> + drag one element onto another = swap their positions

### When pressing over a container's name 

* <kbd>shift</kbd> + <kbd>right click</kbd> = change the container's name

### When pressing on an element's container empty space

* <kbd>right click</kbd> = reveal menu with various actions to take

## ScenesLauncher

### When pressing on a Scene Launcher empty space

* <kbd>right click</kbd> = reveal menu with various actions to take

## MIDI and OSC

Since 4.5.0 there is one **Map** mode for both protocols and one mapping list. The Map icon
appears in the header once a MIDI or an OSC input is wired on the Inputs page.

1. turn on **Map**. Every mappable control shows a badge, and already-mapped controls are badged differently from free ones
2. <kbd>click</kbd> a badge to arm that control (any mouse button). Its badge blinks while it listens, and clicking it again disarms it
3. move a fader, knob or key. It binds to the armed parameter, and learning the same control again moves the mapping rather than stacking a second one
4. edit or remove mappings in **Manage Mappings**: per-mapping **Min** and **Max**, and **jump** or **pickup** takeover. A row marked *(missing)* points at a control that is gone, and **Prune Dead Mappings** clears those
