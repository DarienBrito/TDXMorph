# ParameterMorpher

**Part of the TDXMorph Toolkit**  
Copyright © 2020–2026  
**Author:** [Darien Brito](https://www.darienbrito.com)  
**License:** **PROPRIETARY. Licensed, not sold.**  
**Version:** 4.17.6

> ParameterMorpher is a **commercial** component of the TDXMorph toolkit, governed by the
> ParameterMorpher EULA (see the `LICENSE` operator inside the component). No
> redistribution, resale, sublicensing or sharing. No warranty.
>
> It is **not** MIT licensed and is not distributed from this repository. It is available
> through [Patreon](https://www.patreon.com/c/darienbrito). This page is reference
> documentation only.
>
> The MIT parts embedded inside it, the PresetManager engine and the `Lib/Patterns` library,
> keep their own MIT licence.

---

## 4.17.6

- Maintenance re-export. No change to how the component behaves.

---

## 4.17.5

- **A menu parameter keeps its expression too.** In 4.17.4 the menu element was the one style that still
  replaced an expression with a plain value, so the fix below did not actually reach it.
- **Deleting an element leaves a driven parameter driven.** Removing an element used to reset the
  parameter to a plain value, which threw away the expression or export that 4.17.4 had preserved. A
  parameter that really was a plain value still goes back to the value it had before the element.

---

## 4.17.4

- **An element on a parameter that is already driven keeps it driven.** Creating an element on a
  parameter carrying an expression or an export replaced it with a plain value, so whatever was
  driving that parameter stopped reaching it. The element now leaves the expression or the export in
  place. Driving the element still takes the parameter, which is what it is for. Mappings must be
  re-learned after updating.

---

## 4.17.3

- **A reopened project keeps its morph targets.** After saving a project and opening it again, a
  container had lost the list of parameters it morphs, although every element was still on the
  panel. Randomize did nothing, storing a preset reported that there were no associated parameters
  to make presets from, and adding one element after the reopen made the next Store capture that
  element alone. Containers now keep the list across a save and reopen, and a project already saved
  with the fault repairs itself the first time you open it in this version.

---

## 4.17.2

- **An import no longer empties a container it cannot rebuild.** Importing a container file whose
  source nodes have since been deleted destroyed the elements and left the container empty. The
  import now resolves every parameter the file refers to before it touches anything, and refuses
  the file naming what is missing. Nothing is destroyed.
- **A file imported into a second container lands on that container.** Element order and stored
  preset states were keyed to the container that exported them, so importing elsewhere drove the
  original's elements instead. Both now follow the elements just rebuilt, and an element outside
  the importing container is never written.

---

## 4.17.1

- **MIDI and OSC reach your parameters from the sliders again.** Mapping a knob or fader to a
  Float, Int or XYZ element moved the slider in the panel but never the parameter it controls, and
  learning it again did not help. The mapping now lands on the element itself, the way menus and
  toggles always did, so the knob, the slider and your parameter move together. The Blending slider
  had the same fault and is fixed as well.
- **A parameter driven by an expression or an export can no longer be mapped.** A mapping writes
  values, which would have replaced the expression for good. Clicking such a control in map mode now
  tells you which parameter is driven. This comes from the embedded ControlMapper, now 1.0.5.

---

## 4.17.0

- **Page between Element Containers.** Past two containers the stacked layout gets hard to work
  in, so the header now carries a pager: a single-container view that shows one container at a
  time at full height, stepped with the arrows, picked by name from the menu, or driven from the
  `Page` parameter. The arrows stop at the first and last container rather than wrapping, and both
  are mappable, so a MIDI button can flip pages during a show.
- Paging only changes what you see. A container you have paged away from keeps morphing, keeps its
  timers and keeps answering its mapped controls.
- `Pages mode` is off by default, so an existing project opens exactly as before. While it is off
  the arrows and the menu are greyed out: they are controls of that mode, not a way into it, so
  the toggle is the one way in.

---

## 4.15.2

- **The Support link opens Patreon again.** Six places in the component still pointed at a
  support page that no longer exists, including the one behind the component's own About panel.

---

## 4.15.1

- **Setting a preset outright now records which preset was applied.** Setting rather than morphing
  never updated that record, so the Current Preset readout and the `onPresetCall` callback both kept
  naming the last preset you morphed to. Stepping a preset sequence never advanced the morph counter
  either. Both come from the shared PresetManager engine, so PresetManager 4.2.2 carries the same fix.

---

## 4.15.0

- **An element carries a colour of its own.** Right-click its chip to set one. The colour paints a
  2 px stripe down the element and a 5 px chip, both sized from `Lib/Look` (`Elementstripewidth`,
  `Elementchipsize`). Randomizing no longer overwrites an element's colour.
- **The sixteen colours on offer are derived from the current skin's accent**, so the palette follows
  `Lib/Look.Skin` and a new element seeds from it. The four neutrals (white, grey, dark, black) stay
  put, for marking something uncoloured on purpose.
- **`Lib/Look.Elementcolouring` turns the whole feature off.** Off also retires the picker, since the
  chip is the only way into it. Colours you have already stored survive the switch in either
  direction.
- **Three skins.** `Lib/Look.Skin` picks between **Carbon**, **Verdigris** and **Mono**; Carbon is the
  default. The skin drives every widget colour in the panel, and the element colour palette with it.
- **The morph progress bar is the one under the presets, and only that one.** The second bar is gone.
- **A colour pick writes the exact palette value.** It used to be quantised to 1/255, so a picked
  colour could land a step off the swatch you clicked.
- **Fixed: the Readme button raised an error on every press** on TransportMapper, PresetAnimator and
  PresetGrabber. All three open the repository page now.
- **Fixed: the DIST and CURVE readouts mirror an attached PresetManager again.**

---
## 4.14.2

- **The help sheet opens centred on the monitor TouchDesigner is running on.** It used to open
  at the mouse pointer, against a fixed display index, so on a two-monitor desk it landed under
  the cursor and often on the screen you were not working on. The window now finds
  TouchDesigner's own display every time it opens, and centres there.

---

## 4.14.1

Presets carry their own timing, and the signal row is rebuilt around controls that describe themselves.

- **Each preset stores its own morph time, curve and distribution.** Alongside the per-element
  values it already kept, a preset now records the global settings that were live when you stored
  it. With global timing on, recalling a preset runs on those stored values instead of borrowing
  the timing of whichever element happened to be first in the preset. Two presets can hold
  genuinely different global timings, and neither depends on element order. With global timing
  off, each element still uses its own stored time, exactly as before.
- **Ctrl and right-click on a preset morphs on the global morph time** instead of a fixed one
  second. The old gesture also wrote that 1 into your global morph time and left it there. That
  side effect is gone.
- **Existing presets are safe in both directions.** A preset stored by an older version keeps
  behaving exactly as it did, and a preset saved here still loads in an older version.
- **A Timeline signal now opens on `Project` rather than `Custom`.** `Duration` is what `Custom`
  means, so it no longer appears until you pick `Custom`. A source you set to `Custom` yourself
  stays there.
- Elements got lighter, and the row layout no longer shifts its columns as controls appear and
  disappear.

**Breaking:**

- **A saved MIDI or OSC mapping on a signal row's source or sync control goes dormant.** The
  source menu became three separate menus, one per source, and the sync controls moved one level
  down. Re-map the control once and it stays mapped.
- `Lib/ParametersUI` and `Lib/SignalsUI` merged into `Lib/ElementUI`. Both old names still resolve
  in `CreateFromLibrary()`, though neither is a listed item.

## 4.10.0

A signal can now run on the project timeline instead of on its own clock.

- **New `Timeline` signal source.** Instead of a wave or a pattern, the element plays an
  envelope over a length you set in seconds. `Timelinemode` picks the shape: `Loop` ramps and
  restarts, `Bounce` rises and falls, `Once` runs a single pass and holds.
- **New `Timeline` syncing mode, available to every source.** The signal's phase becomes a
  function of the project timeline position rather than an accumulating clock, so scrubbing the
  playhead scrubs the signal, and an offline render is frame exact.
- **A timeline signal chooses between two lengths and nothing else.** `Project` spans the whole
  project range, `Custom` uses the element's own `Duration` in seconds. The four step and
  transport modes never applied to a timeline signal and are no longer offered for one. LFO and
  Pattern sources keep all six modes.
- `Repetitions` sets how many cycles fit across the project range, which is the unit that means
  something against a fixed length, where a rate in Hz does not.

## 4.8.1

Three fixes, all found by mapping real hardware to the signal controls.

- **Deleting an element now stops its signal.** The element went but the signal kept running
  for the rest of the session, holding the transport clock on. With an LFO source it also
  logged an error every frame; with a pattern source it was silent, which is why it went
  unnoticed. Deleting a whole container was already handled in 4.8.0.
- **Mapping MIDI or OSC to a menu control now works.** Learning a knob onto the signal
  source, the sync mode, the interpolation curve or the random distribution looked like it
  worked and then did nothing: the control moved internally while the visible dropdown and
  the value it feeds stayed put. Sliders were never affected.
- **The signal enable checkbox can be mapped.** Its mapping marker sat on a part of the
  control that carries no value, so a mapping to it had nothing to write.

## 4.8.0

The signal and pattern controls are back inside each element, so several elements can show
their signals at once. They are now **built when you enable a signal and removed when you
disable it**, which is what keeps the size saving: an element with no signal costs the same as
it did in 4.7.x, and only the controls actually on screen exist.

- **Every element with a signal shows its own editor.** In expert mode each element carries a
  `sig` checkbox; ticking it opens that element's signal and pattern rows in place. Comparing
  two elements no longer means expanding one, reading it, then expanding the other.
- **Only what is visible is built.** The sync-step field follows the syncing mode (free running
  builds none at all), and the pattern editor is whichever of the two families your pattern
  type uses. Switching either swaps one control rather than rebuilding the row.
- **Disabling a signal frees the controls and keeps every value.** Your source, wave, pattern
  type, rate, range, smoothing and each pattern type's own values live on the element's
  parameters, not in the controls, so re-enabling restores exactly what you had.

**Sizes to plan a rig around.** An element without a signal is 218 operators. Enabling one adds
106 on the default free-running sequence pattern and at most 140 in the heaviest combination,
so twenty elements with every signal enabled land near 7,900 operators and nothing enabled
stays where it is today. Fifty all-enabled would be around 19,000, which is large enough to
slow the network editor: enable what you use.

**Fixed in the same release:** switching an element's signal source between LFO and Pattern
reset the saved choice for the source you switched to, so an LFO set to Square came back as
Sine. It now keeps both.

**Your existing projects carry forward.** No parameter or stored data changed.

## 4.7.1

Five fixes to the shared signals inspector introduced in 4.7.0.

- **The inspector cut off its own pattern editor.** It ignored the height the container gave it,
  so its controls rendered about three times too tall and the pattern section sat below the fold
  with no way to scroll to it. It also stopped short of the right edge instead of filling the
  width. Both are corrected, so the whole editor is visible.
- **Retargeting left the menus reading the previous element.** Expanding a second element moved
  the inspector, but the signal source, LFO and pattern menus still showed the first element's
  choices. They now follow the element being shown, including on the first retarget in a newly
  created container.
- **Deleting the element the inspector was showing** left it pointing at nothing, which raised
  errors across the panel until another element was expanded.
- **The two sequence fields did not commit what you typed.** A sequence entered in the inspector
  was displayed but never reached the element, so playback kept the previous values.
- **An element's interpolation curve and random distribution menus did not follow their
  parameter.** Changing either by script, by a MIDI or OSC mapping, or through a preset left the
  on-screen menu showing the old choice while the element morphed with the new one.

No parameters or stored data changed, so projects saved on 4.7.0 carry forward untouched.

## 4.7.0

The signal and pattern controls are no longer built into every element. They now live in one
inspector docked at the bottom of the container, and an element's expand toggle chooses which
element it edits. Every interpolatable element drops from 399 operators to 202, and the
component from 4455 to 4270, so a project with twenty elements saves roughly 3,900 operators.

**Your existing projects carry forward.** The parameters holding your signal and pattern setup
have not moved or changed, so nothing needs re-entering. The one difference in use is that two
elements' signals can no longer be shown at the same time, since there is a single inspector
rather than one per element.

The nine pattern scripting methods (`SetPseq` through `SetPbrown`) used to set values by
driving the on-screen widgets. They now write the element's stored pattern data directly, so
they keep working with the controls moved, and each pattern type is given the number of values
it actually takes.

## 4.6.1

- **The pattern editor follows the pattern type again.** Changing an element's pattern type
  stopped switching the editor below it, so picking a type like Pwhite still showed the
  sequence field belonging to the previous type, and any value edited in that state was saved
  into the previous type's slot. Playback was never affected: the signal always played the
  type you selected. Introduced in 4.6.0, where moving the signal engines into one shared
  service removed the part that told the editor which type was selected.
- **If you edited a pattern on 4.6.0**, check the values on each type, since an edit made
  after switching type may have landed on the wrong one.

---

## 4.6.0

- **Signals now run from one shared service instead of a copy inside every element.** A
  project with twenty elements used to run twenty signal engines; there is now a single
  service in the component that elements feed. Every element drops from 441 operators to
  399, and the component from 4574 to 4455.
- **Your existing projects carry forward.** The parameters holding your signal setup
  (source, LFO, pattern, frequency, sync mode, range, smoothing) have not moved or changed,
  the controls behave exactly as before, and nothing needs re-entering. With no signal
  running the service clock stays stopped, so idle cost is unchanged.

---

## 4.5.2

- **Internal quality work only. Nothing in the component behaves differently** and your projects
  are unaffected.

---

## 4.5.1

- **The header map icon is a crosshairs**, replacing the MIDI DIN plug it had inherited. One
  icon covers both protocols, so a MIDI connector was the wrong picture.
- **Fixes MIDI input being ignored.** The mapping service had stopped following the
  component's `MIDI` parameter, so a wired MIDI CHOP never reached it and nothing could be
  learned or routed. OSC was unaffected.

---

## 4.5.0: the MIDI/OSC mapping rebuild (breaking)

Version 4.5.0 replaces the MIDI/OSC mapping subsystem outright. **Mappings made in earlier
versions are not carried forward**, and eight parameters were retired, so any mappings you
already have need to be made again.

What changes for you:

- **One map mode, not two.** The separate MIDI and OSC controls are now a single **Map** icon in
  the header, shown whenever either a MIDI or an OSC input is wired on the Inputs page. Click it,
  click any control, then move a fader, knob or key to bind it. Both protocols share one list.
- **One mapping list.** The `Mappings` page is now **Manage Mappings** and **Clear Mappings**,
  replacing four pulses split across MIDI and OSC sections plus two location toggles. The editor
  shows every mapping in one table.
- **Per-mapping range.** Each mapping carries its own **Min** and **Max**, editable in the list.
  Previously all MIDI mappings shared one global input range.
- **Soft takeover.** Each mapping can be set to `pickup`, which waits until the incoming control
  reaches the parameter's current value before taking over, instead of jumping to wherever the
  fader happens to sit. `jump` is the default and matches the old behaviour.
- **Dead mappings are visible.** If a mapped control is renamed or removed, the row is marked
  `(missing)` in the list rather than silently doing nothing, and can be cleared with **Prune Dead
  Mappings**.
- **Retired parameters:** `Managemidimappings`, `Manageoscmappings`, `Clearmidimappings`,
  `Clearoscmappings`, `Togglelocationmidi`, `Togglelocationosc`, plus the two per-protocol section
  headers on the Mappings page.

Anything can now be a mapping target, not just the built-in controls: a mapping stores a control
and a parameter name, so any custom parameter can be driven.

The rebuild is also most of why the component dropped from 8853 to 4574 operators. The old system
gave every mappable widget its own learn overlay, so an interpolatable element cost 572 operators
in overlays alone; it is now one shared service plus a single marker operator per control.

---

## Overview

The ParameterMorpher is comprised of various components. To access full functionality you
need to interact with both the `extParameterMorpher` class and the `extElementsContainer`
class.

### What changed in 4.x

- **The embedded PresetManager engine is the shared multi-track engine.** ParameterMorpher
  embeds three copies of it, so per-track timing, end modes, curve shapes and preset schema
  v2 are all available here. See [PresetManager](PresetManager.md).
- **The mapping editor is an owned `listCOMP`**, replacing the previous palette TreeListers.
  ParameterMorpher carries no third-party content. As of 4.5.0 it is the shared ControlMapper
  editor described above.
- **The three embedded PresetManager paths editors are owned `ListView` instances.**
- Ships **empty**: no demo content, no stored presets, no bound paths, and **no container**.
  You create the first one yourself, which is the first step under
  [Getting started](#getting-started) below.

---

## Installation

1. Download `ParameterMorpher.tox` from [Patreon](https://www.patreon.com/c/darienbrito).
2. Drag the `.tox` into your TouchDesigner network, or use **File > Import > Component**.
3. Click the component's viewer to open the panel.

There is nothing else to install. The morphing engine is embedded, so ParameterMorpher does
not need a separate PresetManager (unlike [SceneLauncher](SceneLauncher.md), which attaches
an external one).

The licence terms are in the `LICENSE` operator inside the component. Keep the `.tox` in your
project folder or wherever you keep your components; it carries everything it needs.

## Getting started

The component opens **empty**, with no container and no elements. That is intentional: you
build the interface you want rather than deleting a demo first.

### 1. Create a container

Click **Create** in the header, the button whose tooltip reads *Create container*. A
`Container1` appears in the panel and the `Containers` parameter on the **Info** page goes
to 1.

The scripted equivalent, if you are building a project generatively:

```python
op('ParameterMorpher').CreateContainer()
```

Call it again for a second container. Use
`op('ParameterMorpher').ChangeOrientation(horizontal=True)` if you want them side by side
rather than stacked.

### 2. Add parameters

Drag parameters onto the container panel from the network:

- **Drag a single parameter** (from the parameter dialog) onto the container to get one
  slider bound to it. This works for any parameter, built in or custom.
- **Drag a whole node** onto the container (or onto any element row in it) to get one element
  per parameter, but note that this picks up **custom parameters only**. Dropping a node that
  has none, a Noise TOP for example, creates nothing. Drag that node's individual parameters
  instead.

Each element stays bound to its source parameter, so moving the slider drives the original
node directly.

### 3. Store a preset

<kbd>Shift</kbd> + <kbd>Left Click</kbd> an empty preset slot. The current value of every
element in that container is captured into it.

Set the parameters differently and store a second preset. Two presets is the minimum for
morphing to do anything visible.

### 4. Morph

<kbd>Left Click</kbd> a stored preset slot. The container morphs to it over its morph time,
using its interpolation curve.

Two variants are useful while you are still building:

- <kbd>Ctrl</kbd> + <kbd>Left Click</kbd> jumps straight to the preset with no interpolation. Locked
  elements keep their value, as with a plain click.
- <kbd>Ctrl</kbd> + <kbd>Right Click</kbd> morphs over the container's global **Morph Time**
  rather than the time stored in the preset, for a quick transition check.

If you click a preset before adding any elements, the component tells you there is nothing to
apply it to. That is the expected message, not an error.

### 5. Where to go next

<kbd>Right Click</kbd> empty space inside a container for its context menu: *Reset
parameters*, *Clear parameters*, *Rename presets in order* and *Add script*.

The header dropdown builds the companion tools against your containers: *Timers*, *Channels*,
*Grabber*, *Animator* and *Patterns*.

Full key and mouse reference: [SHORTCUTS.md](../SHORTCUTS.md). Everything below this point is
the Python API.

---

## Signals and patterns

Any interpolatable element can drive its parameter continuously, independently of preset
morphing. A signal is either an **LFO** (a periodic wave) or a **pattern** (a stepped sequence
of values). It is chosen per element and runs on the component's own clock.

### Where the controls live

Each element carries its own controls, and they are built on demand. In expert mode an
element shows a `sig` checkbox; enabling it builds that element's signal and pattern rows,
disabling it removes them again. Several elements can therefore show their signals at once.

Only the controls that are visible exist: the sync-step field matches the syncing mode (free
running has none), and the pattern editor is the family your pattern type uses. Changing
either swaps that one control.

The controls are a view, not the engine. Every element runs its signal whether or not its
controls are built, and every value you set lives on the element's parameters below, which is
why disabling and re-enabling restores exactly what you had.

### The model

Signal state lives on the element's `MorphSettings` operator as eighteen custom parameters, and
the engine reads only those. Setting them from Python does exactly what the pane does.

| Parameter | Meaning |
|---|---|
| `Enablesignal` | Turns the element's signal on. Nothing runs while this is off. |
| `Signalsource` | Selects `LFO`, `Pattern` or `Timeline`. |
| `Lfo` | Wave shape, used when the source is `LFO`. |
| `Pattern` | Pattern type, used when the source is `Pattern`. |
| `Frequency` | Rate in Hz, used when the syncing mode is free running. |
| `Syncingmode` | `Internal` (free running), `Manual`, locked to `Beat`, `Bar` or `Sixteenths`, or locked to the project `Timeline`. A `Timeline` source offers `Project` and `Custom` only. Under `Manual` the signal holds still and moves only on `Manualtrigger`. |
| `Manualtrigger` | Advances a pattern by one step. An LFO or timeline signal restarts from its beginning; under `Manual` sync it then holds that value until the next trigger. |
| `Beatfactor`, `Barfactor`, `Sixteenthsfactor` | Multipliers for the locked syncing modes. |
| `Rangex`, `Rangey` | Output range. The raw signal is remapped into it. |
| `Smoothingactive` | Turns output smoothing on. |
| `Smoothrange`, `Smoothlength` | Smoothing amount and window length. |
| `Timelinemode` | Envelope shape for a `Timeline` source: `Loop`, `Bounce` or `Once`. |
| `Duration` | Length of one pass in seconds, used when a timeline signal runs on `Custom`. |
| `Repetitions` | How many cycles fit across the project range under `Timeline` syncing. |

Pattern values are stored separately, as JSON on the element's `Patterndata` parameter. Each
pattern type keeps its own entry, so switching type and switching back preserves what you
entered. Projects made before 4.7.0 migrate automatically the first time they load.

In the pattern editor a sequence can be typed with spaces, commas, semicolons or brackets
(`0.1, 0.5, 1` and `[0.1 0.5 1]` both work). An entry that is not a number is refused with a
message naming it, and a `Pwrand` needs exactly one weight per value.

Enabling a signal from Python has one wrinkle worth knowing. A parameter execute DAT does not fire in the frame its element was created, so setting `Enablesignal` straight after creating an element starts the signal but builds no controls, and the element then has to be toggled twice by hand. Call `EnableSignal(True)` on the element instead: it writes the same parameter and drives the build directly. It is idempotent, so the execute DAT firing a frame later is a no-op.

### Pattern types

Nine methods on an element select a pattern type and set its values in one call. They are the
supported way to script patterns; writing `Patterndata` by hand is not.

| Method | Produces |
|---|---|
| `SetPseq(sequence)` | The sequence in order, looping. |
| `SetPrand(sequence)` | Random picks from the sequence. |
| `SetPxrand(sequence)` | Random picks with no immediate repeats. |
| `SetPshuffle(sequence)` | The sequence shuffled once, then looping. |
| `SetPwrand(sequence, weights)` | Random picks biased by `weights`, one per value. Without `weights`, every value gets equal odds. |
| `SetPseries(start, step, length)` | An arithmetic series. |
| `SetPgeom(start, grow, length)` | A geometric series. |
| `SetPwhite(lo, hi)` | Uniform random values between `lo` and `hi`. |
| `SetPbrown(lo, hi, step)` | Brownian motion bounded by `lo` and `hi`. |

`SetPseries` and `SetPgeom` are finite by definition, so the engine wraps them to repeat
rather than letting the signal run dry and freeze.

### Under the hood

One `SignalEngine` service at the component root drives every element, rather than each
element carrying its own engine. Only active signals occupy the service's table, and its clock
runs only while that table has a row, so a component with no signals enabled costs nothing
when idle.

The service API is documented in [Help/SignalEngine.md](../Help/SignalEngine.md). To resync
every running LFO in a container to a common phase, call `HardSyncLFOs()` on the container. Pattern
signals and Timeline-synced LFOs are left as they are.

---

## exParameterMorpher

The `extParameterMorpher` class is part of the **TDXMorph** system for **TouchDesigner**.  
It manages **container creation**, **library tool instantiation**, and **parameter exposure** within the TDXMorph module.  

This class serves as a **high-level manager** for creating interface elements, preset-related tools, and morphing utilities from the internal library.  
It adheres to a **Model–View–Controller (MVC)** structure, functioning as the *Model*, that is, the logic and data layer independent of UI interactions.

---

## Table of Contents

- [Architecture](#architecture)
- [Initialization](#initialization)
- [Public Methods](#public-methods)
  - [Container Management](#container-management)
  - [Preset Manager Integration](#preset-manager-integration)
  - [UI Layout Control](#ui-layout-control)
  - [Reporting](#reporting)
- [Design Philosophy](#design-philosophy)

---

## Architecture

| **Attribute** | **Description** |
|----------------|-----------------|
| **Class** | `extParameterMorpher` |
| **Role** | Manages creation of containers, preset-related tools, and exposure of morphing data. |
| **Type** | Model / Logic Controller |
| **Design Pattern** | Part of the TDXMorph MVC architecture. |
| **Usage Context** | Used internally to instantiate morphing-related utilities and connect Preset Managers to UI containers. |

---

## Initialization

```python
morpher = op('ParameterMorpher')
```
## Public Methods

### Container Management

- **`GetContainer(n)`**  
  Retrieves a specific container UI by index.  

- **`CreateContainer()`**  
  Creates a new container for slider-based UI elements and links it to the `Elements` component.  

---

### Preset Manager Integration

- **`ExposeTimers(x=0, y=0)`**  
  Creates a composite component exposing **Preset Manager timer channels**.  


- **`ExposeChannels(x=0, y=0)`**  
  Creates a component exposing the **parameter channels** of the UI sliders: one channel per
  element (its value), named after the container and the element's parameter.  

---

### UI Layout Control

- **`ChangeOrientation(horizontal=False)`**  
  Changes the alignment of UI containers between horizontal and vertical layouts.  

- **`ShowPage(n)`**  
  Turns `Pages mode` on and shows the n-th container, 1 based. Returns that container, or
  `None` when none exist.  

- **`NextPage()` / `PreviousPage()`**  
  Steps one container forward or back. Both stop at the ends of the set rather than wrapping, and
  both are inert while `Pages mode` is off.  

- **`PageCount`**  
  How many containers there are, which is the upper bound on the `Page` parameter.  

- **`PageLabel()`**  
  The name and position of the current container, as shown in the header.  

### Reporting

- **`ReportResult(msg, title)`**  
  Displays a popup dialog in TouchDesigner and logs a formatted message to the console.  

---

## Design Philosophy

- **UI Independence:** Logic operates independently of any UI behavior.  
- **Modularity:** Provides an interface layer for morphing and preset operations without visual dependencies.  
- **Automation-Friendly:** Allows automatic generation of morphing tools and managers in project setups.  
- **Extensibility:** Encourages addition of new library components without structural changes.  
- **MVC Integrity:** Respects the TDXMorph Model–View–Controller separation of concerns.

---

## extElementsContainer

---

## Overview

The `extElementsContainer` class manages **groups of UI elements** (sliders, toggles, menus, etc.) in **TDXMorph**, providing a high-level interface between the user interface and the internal `PresetManager`.  

It acts as a **UI-focused controller**, handling element creation, parameter binding, preset management, and synchronization between visual and logical layers.  

This class is primarily used to create and manage **parameter-linked sliders** for morphing and randomization within **TouchDesigner**.

---

## Table of Contents

- [Architecture](#architecture-1)
- [Stored Properties](#stored-properties)
- [Container and UI Management](#container-and-ui-management)
- [Preset Management](#preset-management)
- [Randomization and Morphing](#randomization-and-morphing)
- [Sequence Control](#sequence-control)
- [Utility and Miscellaneous](#utility-and-miscellaneous)
- [Callbacks](#callbacks)
- [Design Philosophy](#design-philosophy-1)
- [Dependencies](#dependencies)

---

## Architecture

| **Attribute** | **Description** |
|----------------|-----------------|
| **Class** | `extElementsContainer` |
| **Role** | Manages groups of morphing UI elements for parameter control. |
| **Type** | UI Controller / Manager |
| **Design Pattern** | Part of the TDXMorph MVC architecture. |
| **Usage Context** | Used to create UI bindings, manage presets, randomization, and morph transitions. |

---

## Stored Properties

| **Property** | **Type** | **Description** |
|---------------|----------|-----------------|
| `BindedPaths` | `dict` | Dictionary mapping cross-check names to binding data for parameters. |
| `containerName` | `str` | Name of the container instance. |
| `NumberOfElements` | `int` | Number of elements contained in the UI container. |
| `GlobalStatus` | `bool` | Whether global morphing and randomization are enabled. |

---

### Container and UI Management

- **`Name`** *(property)*  
  Gets or sets the container’s name.  

- **`Delete()`**  
  Deletes all UI elements and destroys the container panel.  

- **`UpdateSize(sizeLimit=720)`**  
  Adjusts the height of the container based on its content.  

- **`HardSyncLFOs()`**  
  Restarts the phase of every running LFO signal in the container, so they move in step.
  Pattern signals and Timeline-synced LFOs are left as they are.  

- **`ChangePresetsNum(newVal)`**  
  Changes the number of presets available in the container.  

- **`GetBindedKeys()`**  
  Returns a list of all keys from the `BindedPaths` dictionary.  

- **`GetPresetManager()`**  
  Returns the internal `PresetManager` component reference.  

- **`GetElementsValues()`**  
  Returns a dictionary of element values and ranges.  

- **`CreateElement(source, parameter, dataSource='parameter', customName=None)`**  
  Creates and binds a new UI element linked to a parameter or preset.  
  Handles the correct UI type allocation, binding, and reference registration.  

- **`AddScript()`**  
  Creates a special script element that executes custom code when triggered.  

- **`ClearParameters()`**  
  Removes all UI parameters after confirmation. Their stored values leave every preset too, and
  the confirmation says how many presets that affects.  

- **`ResetParameters()`**  
  Resets all UI elements to their default stored values. A slider's range resets on the slider
  and its signal settings too, and a Menu or StrMenu element keeps one slot per item.  

- **`RenamePresetsOrder()`**  
  Renames and reorders presets based on current UI order. The selected preset and the blend
  endpoints follow their presets to the new names.  

- **`ExpertMode(value)`**  
  Enables or disables “Expert Mode” for advanced control settings.  

---

### Preset Management

- **`ExportPresetsJSON()`**  
  Exports all presets, bindings, and element data to a `.json` file.  
  Used for saving full configuration states including UI bindings.  

- **`ImportPresetsJSON()`**  
  Imports presets and bindings from a `.json` file, reconstructing the UI and restoring order.  

- **`ClearPresets(overwriteWarning=True)`**  
  Clears all presets in both macro and micro (local) managers.  

---

### Randomization and Morphing

- **`SyncClocks()`**  
  Synchronizes morph timers across all linked elements.  

- **`SetRandom(mode=None)`**  
  Randomizes parameter values globally or per element.  

- **`MorphRandom(mode=None)`**  
  Performs a morph transition toward randomized target values.  

- **`MorphAll(mode=None)`**  
  Morphs every element at once, each on its own timing and curve from its `MorphSettings`. This is the multi-track counterpart of `MorphRandom`, and it behaves single-track unless the engine's `Multitrack` gate is on.  

- **`MorphGroup(tag, mode=None)`**  
  Morphs only the interpolatable elements whose `MorphSettings.Group` matches `tag`, each to a fresh random target on its own timing and curve. It forces the engine's per-track gate on, because a group is inherently multi-track, and elements outside the group keep their absolute start, so a morph already in flight is undisturbed.  

---

### Sequence Control

- **`PresetsSequence()`**  
  Traverses the presets in the order their buttons show, across all linked elements.  

---

### Utility and Miscellaneous

- **`Interpolation`** *(property)*  
  Gets or sets the interpolation curve used for morphing.  

- **`Distribution`** *(property)*  
  Gets or sets the random distribution type.  

- **`GlobalTime`** *(property)*  
  Toggles global vs local morph timing control.  

- **`Automorph` / `Automorphs`** *(property)*  
  Controls automatic morph playback and its count.  

- **`Time`** *(property)*  
  Gets or sets the morphing duration.  

- **`NumberOfPresets`** *(property)*  
  Gets or sets the total number of available presets.  

- **`StorePreset(slot)`**  
  Stores the current preset into a numbered slot. A slot is a whole number from 1 up to
  `NumberOfPresets` (`2.0` counts as 2); anything else gives a warning.  

- **`SetPreset(slot)`**  
  Loads and activates a stored preset. Same slot rule as `StorePreset`.  

- **`DeletePreset(slot)`**  
  Deletes a stored preset by index (a whole number from 1; a slot above the current count can
  still be deleted).  

- **`GetElement(elementNum)`**  
  Returns a specific UI element by its numeric order.  

- **`Freeze(value)`**  
  Locks or unlocks the entire UI, preventing accidental changes.  

---

## Callbacks

The following methods execute when connected callback scripts are defined:

- **`OnMorphingStart(manager, morphingType, presetName)`**  
  Triggered when a morphing process starts.

- **`OnMorphingEnd(manager, morphingType, presetName)`**  
  Triggered when a morphing process ends, including when Stop ends it.

- **`OnPresetCall(manager, morphingType, presetName)`**  
  Triggered when a preset is invoked manually or programmatically.

Each one calls the matching function in the DAT set on `Elementscontainercallbacks`, passing the
container first:

```python
def onMorphingStart(elementsContainer, presetManager, morphingType, presetName): ...
def onMorphingEnd(elementsContainer, presetManager, morphingType, presetName): ...
def onPresetCall(elementsContainer, presetManager, morphingType, presetName): ...
```

A blank `Elementscontainercallbacks`, or a DAT without the function, is ignored.

---

## Design Philosophy

- **UI-Driven Logic:** Designed for dynamic UI creation and interaction with parameters.  
- **Modularity:** Each container acts independently but can synchronize with others.  
- **Non-Destructive:** Parameters can be freely created, rebound, or deleted. 
- **Automation-Friendly:** Supports scripting, callback hooks, and sequenced morphing.  
- **MVC Compliance:** Acts as a **Controller** in the TDXMorph architecture, linking UI (View) and PresetManager (Model).

---

## Dependencies

### TouchDesigner Components
- `PresetManager`
- `PresetMorpher`
- `Menus`
- `Elements`
- `Parameters`
- `TDResources`
- `Lib/Elements/*`

### Python Libraries
- `TDStoreTools.StorageManager`
- `json`

---
