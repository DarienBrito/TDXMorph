# ParameterTimeline

**Part of the TDXMorph Toolkit**  
Copyright © 2026  
**Author:** [Darien Brito](https://www.darienbrito.com)  
**License:** **PROPRIETARY. Licensed, not sold.**  
**Version:** 1.4.0

> **Coming soon.** ParameterTimeline is not released yet. This page documents the component as it stands
> and may change before release.

> ParameterTimeline is a **commercial** component of the TDXMorph toolkit, governed by the
> ParameterTimeline EULA (see the `LICENSE` operator inside the component, or About > License).
> Use it in anything you make, client work included; credit appreciated, not required. Hand it over
> only inside a finished project: your client runs it there, with no reuse, sharing or resale.
> No sharing or resale of the component, alone or built into tools of your own. No warranty.
>
> It is **not** MIT licensed and is not distributed from this repository. It is available
> through [Patreon](https://www.patreon.com/darienbrito). This page is reference
> documentation only.

---

## Overview

`ParameterTimeline` drives parameters from **keys placed on a timeline**, and plays that timeline from **its own transport** (play, pause, stop, loop) or from **an external clock**: Ableton Live's transport through Nexum, any CHOP that carries a position, or a `Position` parameter you animate yourself. It drives the custom parameters of one COMP, the **Target**, and any parameter of an operator inside it that you add.

You make keys by **posing**: set the parameters by hand, then store a key. A key holds a value for the parameters you moved, a transition curve into it, and optionally a hard cut. Keys can be anchored to **markers**, named positions such as the sections of a song, so they move with the song when the arrangement changes.

The timeline is **baked** when you edit it and only looked up while it plays. The output is a pure function of the clock position: jumping, looping or scrubbing in Live lands on the right pose at once. Nothing cooks while the clock stands still, and only parameters whose value changed are written.

While it plays you can take a parameter over: move it by hand, from a MIDI controller or from a script, and it stops following while the rest keep playing. **Back to Timeline** hands it back, gliding if you like, and **Record** turns what you played into keys.

Typical uses: camera moves locked to a song, lighting or effect parameters that follow the arrangement, and anything else that should be the same every time a section plays.

---

## Table of Contents

- [Architecture](#architecture)
- [Quick start](#quick-start)
- [Playback](#playback)
- [Following Ableton Live with Nexum](#following-ableton-live-with-nexum)
- [Camera work](#camera-work)
- [The model](#the-model)
- [The editor](#the-editor)
- [Tempo and units](#tempo-and-units)
- [Output](#output)
- [Taking over while it plays](#taking-over-while-it-plays)
- [Status messages](#status-messages)
- [Custom parameters](#custom-parameters)
- [Saving and loading timelines](#saving-and-loading-timelines)
- [Public methods](#public-methods)

---

## Architecture

| **Attribute** | **Description** |
|---|---|
| **Class** | `extParameterTimeline` |
| **Role** | Keyed poses on a beat or seconds axis, baked on edit, evaluated from its own transport or an external clock. |
| **Drives** | The custom parameters of the COMP set in `Target`, and any parameter of an operator inside it that you add. |
| **Embeds** | The PresetManager curve library and `ListView` (both MIT). |
| **License** | Proprietary (EULA) |

The timeline is stored inside the component and saved with your project. Editing it (a store, a drag, a typed value, a marker table change) rebakes it. Playback evaluates the baked table at the clock position and writes the parameters that changed.

---

## Quick start

This works without Live: the timeline plays from its own transport (**Clock Source** `transport`, the default).

1. Make a COMP with a few custom float parameters, for example a base called `rig` with `Tx`, `Ty` and `Rx`. Those parameters are what the timeline will drive.
2. Drop `ParameterTimeline.tox` next to it and set **Target** to `rig`. The **Status** parameter says what is still missing at each step; follow it until it is empty.
3. On the Keys page, turn **Edit** on. Set `rig`'s parameters to a first pose, set **Position** to `0` and pulse **Store Key**.
4. Set a second pose, set **Position** to `8` and pulse **Store Key** again.
5. Turn **Edit** off, set **Position** to `0` and turn **Play** on (Transport page; **Space** in the editor). `rig` moves from the first pose to the second. **Stop** goes back to the start.

Pulse **Open Editor** to see the timeline: your channels on the left, their curves in a graph with the keys on top, and a card for the selected key. Hover over any icon or control and a tooltip says what it does.

The first time you bind a Target or store a key, the timeline takes its **channels** from the target: every writable number, toggle and menu custom parameter that **Channels** matches (`*`, all of them, by default). Floats interpolate; integers, toggles and menus step. To add more, drag parameters from any operator inside the Target onto the editor window.

To drive only part of a COMP, set **Channels** to a pattern of parameter or group names, for example `Hue Size Pos` (`Pos` takes `Posx`, `Posy` and `Posz`). The rest of the COMP stays yours, for MIDI, another timeline or your hands. Changing it later adds the parameters it now matches and removes the ones it no longer matches, with their keyed values; parameters you dragged in stay.

**Store Key** keys only what you moved since the timeline last wrote it (and anything not keyed yet), so a new key can hold just one parameter. Turn **Key All Channels** on to key every channel each time.

---

## Playback

**Clock Source** picks what plays the timeline:

| **Clock Source** | **Plays it** |
|---|---|
| `transport` (the default) | The timeline's own transport: **Play** (off = pause, which holds the spot), **Stop** (back to the start), **Loop** between **Loop Start** and **Loop End**, **Rate**. Through the tempo map. |
| `chop` | A CHOP carrying a position: Ableton Live through Nexum (below), timecode, another timeline, any CHOP. |
| `position` | The **Position** parameter: animate it, script it, or let another tool drive it. |

In the editor, **Space** plays or pauses, **Enter** stops and **Ctrl+L** turns the loop on or off (see [The editor](#the-editor)). Pause writes the spot it stopped on into **Position**, and Stop sets **Position** to the start, so a Store Key or an edit after either happens where the playhead shows.

**Preview in Edit mode.** With **Edit** on, Play previews the timeline whatever the Clock Source, so you can watch your edit without starting Live. A parameter you moved by hand while posing stays where you put it (it shows as overridden). Turning **Edit** off ends the preview and the Clock Source takes over again. Outside Edit, Play works only with Clock Source `transport`; the status bar says so otherwise.

**Following another timeline.** `out_transport`, the component's second output, carries `beat`, `time` (seconds), `tempo` and `playing` (1 while the transport runs) for wherever this timeline is. It has the same channels as Nexum's `out_transport`, so another ParameterTimeline can use it as its **Clock CHOP**, and one transport plays several timelines.

---

## Following Ableton Live with Nexum

Nexum sends Live's transport, tempo and locators to TouchDesigner. With its `NexumReceiver` in the project:

| **Parameter** | **Set it to** |
|---|---|
| **Clock Source** | `chop` |
| **Clock CHOP** | the receiver's `out_transport` |
| **Clock Channel** | `beat` (the default) |
| **Clock Unit** | `beats` (the default) |
| **Tempo Channel** | `tempo` (the default) |
| **Markers DAT** | the receiver's `cues` table |

`out_transport` carries Live's song position in beats, so the timeline follows playback, jumps, loops and scrubbing. The `cues` table lists Live's **locators** (`idx`, `name`, `beat`). Name your locators after the sections of the song; keys stored after a locator anchor to it, and moving the locator in Live moves those keys. Locators without a name are ignored.

To store keys at a spot in the song, turn **Edit** on (playback stops writing to the target), set **Position** to the beat you want, pose the target and pulse **Store Key**. Or double-click the editor's graph at that spot.

The timeline follows Live's **arrangement** position. It suits arranged pieces and cued sections. For improvised Session view clip launching, where the song position does not track the form, a preset launcher such as SceneLauncher fits better.

---

## Camera work

Two ways in. **Direct:** put the camera inside the Target COMP and drag its `t`, `r` and `fov` parameters onto the editor window; a dragged `tx ty tz` becomes one lane group. **Through a rig**, when you want named controls:

1. Make a base COMP, for example `rig`, with one custom float per camera value: `Tx`, `Ty`, `Tz`, `Rx`, `Ry`, `Rz`, `Fov`.
2. On the camera, reference them with expressions: `tx` = `op('rig').par.Tx`, and so on.
3. Set ParameterTimeline's **Target** to `rig`.

Rotations need one more step. A plain number goes from 350 to 10 the long way, back through 180. Put the rotation channels in **Angle Channels**, for example `Rx Ry Rz` or `R*`, and they take the short way round: 350, 360, 370 (which the camera reads as 10). Set it before or after storing keys; keys stored before are adjusted, in order, so each takes the short way from the one before.

The same rig works for orbit setups: drive `Azimuth`, `Elevation` and `Distance` custom parameters and let expressions place the camera. Put `Azimuth` in Angle Channels.

---

## The model

**Keys.** A key is a set of values at a position. It holds any subset of the channels: each channel runs through the keys that hold it and ignores the others. Each key has:

| **Field** | **Meaning** |
|---|---|
| anchor | A marker plus an offset (`Break+8`), or an absolute position when no marker comes before it. |
| curve | The transition curve into this key, from the PresetManager set: `Linear`, `Scurve`, `Easein`, `Easeout`, `Sine` and others. |
| length | How long the transition into this key lasts, ending on the key. `0` = the whole span from the previous key. |
| per-channel curve / length | Optional, per channel: that channel's own curve or length into this key. Empty = the key's. |
| cut | A hard cut: the channels this key holds jump at it instead of moving into it. |
| values | One pose per variant. |

**Markers.** A table of named positions. Columns are read by header when the first row has a `name` column and a `position`, `beat`, `beats`, `time` or `seconds` column; otherwise the first column is the name and the second the position. A key stored after a marker anchors to the last marker at or before it, so moving the marker moves the key. A key whose marker is no longer in the table is left out of playback, and Status names the marker.

**Variants.** One list of keys, several takes of values. A key holds values per variant and may be missing from a variant. **Variant** picks the take that plays and the one you edit. A new timeline has one variant, `main`. Type a new name into **Variant** to make another: it starts as a copy of the variant you were on, so it plays the same until you store into it.

**Channels.** One per driven parameter: a custom parameter of the Target, or any number, toggle or menu parameter of an operator inside it (a camera's `tx`, a constant CHOP's value). String, string-menu, operator, Python and pulse parameters cannot be keyed. A parameter under an expression or an export is **driven**: the timeline never writes it, and its values still reach the output CHOP. Each channel is of one of three kinds:

| **Kind** | **Behaviour** |
|---|---|
| `linear` | Interpolates along the curve between keys. |
| `angle` | Like linear, the short way round (see [Camera work](#camera-work)). |
| `step` | Holds each key's value until the next key. Integers, toggles and menus. |

Before a channel's first key and after its last, its value holds. Channels belong to a **group** whose smoothing (a Gaussian, in timeline units) rounds off the corners; a new timeline has one group, `main`, with no smoothing.

**Envelopes.** Optional, per variant: a `rise` span blends from the first key's pose into the motion, and a `settle` span blends onto the last pose. They come with an imported timeline.

---

## The editor

Pulse **Open Editor**. The editor opens in its own window, which you can resize.

- **Top bar:** Edit, play, stop, loop, the key buttons, variants, snap, record and settings.
- **Left:** the parameters this timeline moves, one row each, with their current value.
- **Overview** (top right): the whole timeline in one strip. The box shows the part you are zoomed into. Its keys are only shown there; select and move keys in the graph.
- **Loop lane:** the thin strip under the overview.
- **Graph:** each parameter is a line, and each diamond along the top is a key. The coloured line is the one you are working on; click a name on the left to pick another.
- **Ruler** (bottom): minutes and seconds; in bars and beats, bars and beats with minutes and seconds under them.
- **Status bar** (very bottom): tells you when something is wrong, and lists the mouse moves for the graph. The version is on its right. Hover over any button for a short tip.

What you see on the graph is exactly what plays.

### Playing

| **Do this** | **To** |
|---|---|
| **Space** | Play or pause. |
| **Enter** | Stop and go back to the start. |
| **Ctrl+L** | Turn the loop on or off. |
| Drag in the loop lane | Set the loop. Drag its ends to resize it, its middle to move it. Double-click the lane to turn the loop on or off. |
| Click the ruler or an empty spot on the graph | Jump there. |
| Drag the ruler | Scrub. |
| Drag the playhead's position label (top of the graph) | Move the playhead from where it is. |

While it plays, the graph scrolls along with it. If you scroll somewhere else, it stays where you put it until you press play again.

### Keys

A key remembers what your parameters look like at one moment. The timeline moves smoothly from one key to the next.

| **Do this** | **To** |
|---|---|
| Double-click the graph | Store a key there, with your parameters as they are now. |
| Click a key | Select it and jump to it. |
| Drag a key | Move it. The lines follow as you drag. |
| **Ctrl**+drag a key | Copy it. |
| **Delete** or **Backspace** | Delete the selected key. |
| Right-click a key | More options: delete it, make it a hard cut, change its curve. |
| The arrows in the top bar | Go to the previous or next key. |

**Several keys at once.**

| **Do this** | **To** |
|---|---|
| Drag across empty space in the graph | Select every key in that stretch of time. |
| **Shift**+drag, or **Shift**+click a key | Add to the selection (Shift+click a selected key takes it out). |
| Drag one of the selected keys, or a point of one on the curve | Move them all together in time; **Ctrl**+drag copies them. |
| Drag a selected point up or down | Raise or lower that parameter by the same amount in every selected key (**Shift** keeps it to one direction). |
| **Delete** or **Backspace** | Delete them all. |
| Right-click inside the selection | Delete them, or select none. |
| **Ctrl+A** | Select every key. |
| Click a single key, or empty space | Select just that key, or nothing. |

### One parameter at a time

| **Do this** | **To** |
|---|---|
| **Ctrl**+click a line or band | Add a point to just that parameter. The line keeps its shape. Integers, toggles and menus have a band under the lines instead of a line: a stepped line over one bar per key, each bar labelled with its value (`off` / `on` for a toggle, the item's label for a menu). |
| Drag a point sideways | Move it in time. |
| Drag a point up or down | Change its value. In a band it steps: one step per band height, up turns a toggle on or picks the next menu item. |
| Hold **Shift** while dragging | Move only in time, or only in value. |
| Right-click a point | Remove it, or give that parameter its own curve into the key. On a toggle's band: **Turn on** / **Turn off**; on a menu's band: pick its item. |

### Snapping to the grid

Keys, points and the loop snap to the grid. Hold **Alt** while dragging to place them freely. The magnet in the top bar turns snapping off, and the menu next to it picks the grid size. Right-click empty space in the graph for the same menu.

| **Do this** | **To** |
|---|---|
| **Ctrl+1** / **Ctrl+2** | Finer / wider grid. |
| **Ctrl+3** | Triplets on or off. |
| **Ctrl+4** | Snapping on or off. |
| **Ctrl+5** | Switch between the automatic grid and the last fixed one. |

### Zooming

| **Do this** | **To** |
|---|---|
| Mouse wheel over the graph or the overview | Zoom in or out where the mouse is. |
| Drag the box in the overview | Move along the timeline. Drag its edges to zoom. |
| **Fit all** (top bar, after the grid menu) | Show the whole timeline. After a long playback this also removes the empty space it ran past. |
| **Fit key** (next to it) | Zoom to the selected key. |

### The key card

Click the card button in the top bar, or press **Ctrl+K**, to show the selected key's details next to it. Turn it off when it gets in the way. The card shows:

- the marker the key follows, and how far after it
- the shape and length of the move into the key
- which variants have this key
- the key's values: double-click one to type a new number

### The parameter list

- **Add parameters** by dragging them from your project onto the editor window.
- The **eye** shows or hides a parameter's line.
- Click a **name** to make it the coloured line.
- The **×** removes a parameter. Click it twice to confirm.
- A **red row** lost its parameter (renamed or deleted). Drop the right parameter on it to reconnect it.
- **Tempo** sits at the bottom of the list. Its eye shows or hides the tempo line.

### Taking over by hand

If you move a parameter yourself while the timeline plays, that parameter stops following the timeline and shows a back arrow. Click the arrow to hand it back. The back arrow in the top bar hands back all of them.

Turn **Edit** on to pose your scene by hand: the timeline stops moving it, and play still previews. Turn Edit off and the timeline takes over again.

**Record:** turn it on, play, and move parameters by hand. When you turn it off, your moves become keys.

### Variants and colours

**Variants** are different versions of the same timeline, like takes. The menu in the top bar switches between them, adds one or renames one.

The **palette** button changes the editor's colours.

---

## Tempo and units

A timeline counts in **seconds** (the default for a new one) or in **bars and beats**. Switch in the grid menu (the menu next to the magnet, or right-click empty graph space): **Time in seconds** / **Time in bars and beats**. Every key keeps its time when you switch. Going to bars and beats, you can also snap each key to its nearest beat. In seconds the ruler, the grid and the clock show minutes and seconds, and the tempo row is hidden.

The clock and the markers each have their own unit (**Clock Unit**, **Marker Unit**); when one differs from the timeline's, it is converted through the tempo.

**Tempo map.** An optional table, one row per tempo change: `beat`, `bpm`, and `hold` (the tempo stays until the next row) or `ramp` (it moves linearly to the next row). Without a map, **BPM** is used.

**Learn.** With a beats clock that has a tempo channel (Nexum's `out_transport` does), turn **Learn Tempo Map** on, play the song through the tempo changes, and turn it off. The recorded tempo is simplified into `ramp` rows ending in a `hold` row and written into the **Tempo Map DAT** (header `beat bpm shape`), replacing its rows inside the recorded range.

**Tempo Drift** shows the clock's tempo minus the map's at the current beat. Near 0 means the map matches the song. It stays 0 without a tempo map, and updates only from a beats clock CHOP while Edit is off.

---

## Output

With **Write Pars** on (the default), the evaluated values are written onto the target's parameters. Only parameters whose value changed are written, and nothing is written while **Edit** is on.

The same values are on the component's output CHOP, `out1`, one channel per keyed channel. Turn **Write Pars** off to use the CHOP instead, for example to combine the timeline with other motion before it reaches the target. `out1` always carries the timeline's values, whatever is overridden or gliding.

The second output, `out_transport`, carries the playhead: `beat`, `time`, `tempo` and `playing` (see [Playback](#playback)).

---

## Taking over while it plays

**Override.** Move a parameter the timeline writes from anywhere else (by hand, from a controller mapped with ControlMapper, from a script) and that channel stops following; the others keep playing. **Overridden** lists the channels that are. Storing a key releases what it stores.

**Back to Timeline** (the pulse) hands every overridden channel back; the back arrow on a channel's row in the editor hands back that channel only. With **Glide** above 0, a float eases from where you left it to the timeline's value over that many seconds instead of jumping; touch it during the glide and it is overridden again. Integers, toggles and menus always snap, and nothing glides while **Write Pars** is off or **Edit** is on. Turning **Edit** off snaps too.

**Record.** Turn **Record** on and play: every channel you move from outside while the clock runs is sampled, starting from where it was just before you touched it. Turn Record off and the takes become keys in the active variant, holding only those channels, thinned so that the motion stays within **Record Tolerance** (in the parameter's units) of what you played. The recorded channels then follow the timeline again. Nothing is stored for a channel you did not move, unless it was already overridden when Record started.

---

## Status messages

**Status** (Timeline page) and the editor's status bar say what is wrong, and are empty when nothing is.

| **Message** | **Fix** |
|---|---|
| `no Target: ...` | Set **Target** to the COMP whose custom parameters the timeline drives. |
| `Target has no par ...` | The target lacks a parameter the timeline drives. Add it, or point Target at the right COMP. `op:par` = a parameter of an operator inside the target that is gone or renamed: drop the parameter on its red row in the editor to relink it. |
| `not added: ...` | A dropped parameter that cannot be keyed: read only, not a number, toggle or menu, or one of the timeline's own. One from outside the target brings up **Widen the Target**: **Widen** moves the Target to the COMP that holds both and adds it. |
| `Target is this ParameterTimeline: ...` | A timeline cannot drive itself. Set **Target** to another COMP. |
| `not relinked: ...` | A relink needs a number for a number channel and a toggle, menu or integer for a stepped one, and a parameter that is not already a channel. |
| `no Clock CHOP: ...` | Set **Clock CHOP**, or set **Clock Source** to `transport` or `position`. |
| `Play: this timeline follows ...` | Play runs the built-in transport. Set **Clock Source** to `transport`, or turn **Edit** on to preview. |
| `Clock CHOP has no channel ...` | Set **Clock Channel** to a channel the CHOP has. |
| `...: nothing to key ...` | The target has no number, toggle or menu custom parameter. The prefix names what stored: `Store Key` (the pulse), `button` (the editor's Store) or `store` (a double-click on a lane). |
| `...: nothing moved ...` | Nothing changed since the timeline wrote it. Move a parameter, or turn on **Key All Channels**. Same prefixes. |
| `no marker named ...`, `no curve named ...` | A typed name that does not exist. Use the menu. |
| `bake failed: ...` | The timeline could not be baked; the message says why. |
| `learn: ...` | Learn needs a Tempo Map DAT and a playing beats clock with a tempo channel. |

A problem with an edit clears on the next successful edit. A clock problem clears once the clock resolves.

---

## Custom parameters

Every parameter has a tooltip in the parameter dialog.

### Timeline page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Target` | COMP | The COMP whose parameters the timeline drives. Changing it keeps every channel whose operator is inside the new one (point it at a parent to widen it); the timeline's own parameters never key. |
| `Channels` | Str | The Target's custom parameters to drive, as a pattern of parameter or group names (`Hue Size Pos`). `*` = all. |
| `Variant` | Menu | The take that plays, and the one Store and the editor work on. Type a new name to make a variant. |
| `Markers` | DAT | Named positions that keys anchor to. |
| `Markerunit` | Menu | `beats` or `seconds`. |
| `Anglechannels` | Str | Channels that interpolate as angles, as a pattern (`Rx Ry Rz`, `R*`). |
| `Writepars` | Toggle | Write the values onto the target. Off: use the output CHOP. |
| `Status` | Str | What is wrong right now, if anything. Read only. |
| `Rebake` | Pulse | Rebuild the baked timeline. Edits and table changes do this by themselves. |

### Keys page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Keyseditor` | Pulse | Open the editor window. |
| `Edit` | Toggle | Hold the target for posing: the clock stops writing, Position drives, and selecting a key poses the target. Turning it on poses the target at Position; off resumes playback. **Play** previews while it is on. |
| `Grid` | Menu | The grid keys, points and the playhead snap to: **Adaptive** (follows the zoom: a note step, or seconds on a seconds timeline), a note value (`1 Bar` to `1/32`; `1/4` = one beat) or seconds (`1 s` to `0.1 s`, through the tempo map). |
| `Triplet` | Toggle | Note steps times 2/3. |
| `Snap` | Toggle | Snap to the Grid. Off, or **Alt** held in the editor, places freely. |
| `Selectedkey` | Int | The selected key's id. Setting it selects that key. |
| `Keyall` | Toggle | Store and Update key every channel. Off (the default): only the channels you moved, plus any not keyed yet. |
| `Storekey` | Pulse | Store what you moved as a key at the playhead (the running transport; else Position in Edit mode or with Clock Source `transport` or `position`; else the Clock CHOP), in the active variant. A key already there is updated. |
| `Updatekey` | Pulse | Merge what you moved (every channel with Key All Channels) into the selected key's values in the active variant. |
| `Deletekey` | Pulse | Remove the selected key from the active variant. |
| `Prevkey`, `Nextkey` | Pulse | Select the previous or next key of the active variant. |
| `Loadpose` | Pulse | Pose the target with the timeline's values at Position. |
| `Overridden` | Str | The channels that stopped following because something else moved them. Read only. |
| `Glide` | Float | Seconds that Back to Timeline takes to ease a moved number back. `0` = snap. |
| `Backtotimeline` | Pulse | Hand every overridden channel back to the timeline. |
| `Record` | Toggle | Sample the channels you move while the clock runs; turning it off stores them as keys. |
| `Recordtolerance` | Float | How far the recorded keys may stray from what you played, in the parameter's units. Larger = fewer keys. |

### Transport page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Play` | Toggle | Play from Position. Off = pause: Position takes the spot it stopped on. Works with Clock Source `transport`, or in Edit mode as a preview. |
| `Stop` | Pulse | Stop and go back to the start of the timeline. |
| `Loop` | Toggle | Loop between Loop Start and Loop End. |
| `Loopstart`, `Loopend` | Float | The loop range, in the timeline's unit. The editor's loop lane sets them. |
| `Rate` | Float | Playback speed. `1` = the tempo map's speed. |

### Clock page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Clocksource` | Menu | `transport` (the default): the timeline's own transport. `chop`: follow the Clock CHOP. `position`: follow the Position parameter. |
| `Clock` | CHOP | The CHOP carrying the clock, e.g. Nexum's `out_transport`. |
| `Clockchannel` | Str | The channel carrying the position. Default `beat`. |
| `Clockunit` | Menu | `beats` or `seconds`. |
| `Position` | Float | The position when Clock Source is `position`, and in Edit mode. With `transport`, where Play starts and where Pause and Stop leave the playhead. |
| `Tempomap` | DAT | Optional tempo changes: `beat`, `bpm`, `hold` or `ramp`. |
| `Bpm` | Float | Tempo used when there is no tempo map. |
| `Tempochannel` | Str | The clock's tempo channel, for Learn and Tempo Drift. Default `tempo`. |
| `Learn` | Toggle | Record the clock's tempo; turning it off writes the Tempo Map DAT. |
| `Tempodrift` | Float | Clock tempo minus map tempo at the current beat. Read only. |

### About page

The `Readme` (this page), `Support` (labelled Patreon) and `Website` pulse buttons, then `Author` and `Version`, which are **read only**.

---

## Saving and loading timelines

The timeline is stored inside the component and saved with your project. To move it between projects or keep versions of it, read and write it as JSON:

```python
import json
timeline = op('ParameterTimeline').Timeline
open('song.json', 'w').write(json.dumps(timeline, indent=1))

op('ParameterTimeline').ImportFile('song.json')
```

A timeline is a dict:

| **Key** | **Holds** |
|---|---|
| `unit` | `beats` or `seconds`. |
| `step` | Bake resolution in timeline units (default `0.0625`). |
| `channels` | `{par, group, kind, extrapolate, envelope}` per channel, plus `op` and `name` for a parameter of an operator inside the target (`op` relative to it) and `tuplet` for a lane group. `par` is the channel's id and its `out1` name. `kind` is `linear`, `angle` or `step`. |
| `groups` | Group name to smoothing (Gaussian sigma in timeline units, `0` = none). |
| `variants` | The variant names, in menu order. |
| `envelopes` | Per variant, optional `rise` and `settle` spans, each a pair of anchors. |
| `keys` | `{id, anchor, cut, transition: {curve, length}, values: {variant: [{paramName, value, ...}]}}` per key, plus optional `transitions: {channel: {curve, length}}`. A key's values hold only the channels it keys; `paramName` is the channel id. |

An anchor is `{marker, offset}` or `{position}`. Importing a timeline replaces the current one; its channels are kept as they are, and **Angle Channels** shows its angle channels.

---

## Public methods

```python
StoreKey(position=None, snap=True)
```
Store what you moved (every channel with Key All Channels) as a key at `position` in the active variant, and select it. Default position: the playhead (the running transport; else Position in Edit mode or with Clock Source `transport` or `position`; else the Clock CHOP), on the grid unless `snap` is False. A key already at that spot is updated. Returns the key id. Raises `ValueError` when there is no Target, the target has nothing to key, a channel's parameter is missing, or nothing moved.

```python
UpdateKey(keyId=None)
DeleteKey(keyId=None, allVariants=False)
```
Merge what you moved (every channel with Key All Channels) into a key's values in the active variant, or remove them; `UpdateKey` raises `ValueError` when nothing moved. Default: the selected key. A key goes once no variant has values left, or at once with `allVariants`.

```python
AddChannel(par, group='main', tuplet=None)
RemoveChannel(channelId)
RemoveChannels(channelIds)
Relink(channelId, par)
```
Add a parameter of the target or of an operator inside it as a channel (returns its id: the parameter name, or `<operator name>_<par>` inside the target, with `_2`, `_3` on a clash; a parameter that is already a channel returns that id), drop a channel (or several in one edit) and its stored values, or point a channel at another parameter, its keys kept. `AddChannel` and `Relink` raise `ValueError` for a parameter that cannot be keyed.

```python
SetChannelTransition(keyId, channelId, field, value)
MoveChannelValue(keyId, channelId, position, snap=True)
```
Set one channel's own `curve` or `length` into a key (`None`, `''` or `'(key)'` clears it), or move one channel's value from a key to the key at `position` (made when there is none). `MoveChannelValue` returns the key that holds the value now.

```python
WidenTarget(comp, outside=())
```
Set the Target to `comp`, every channel re-expressed so it keeps its parameter, then add the `(par, laneGroup)` pairs in `outside` (laneGroup `None` for a single parameter). Returns their channel ids. A drop from outside the Target offers this.

```python
AddChannelKey(channelId, position, value=None, snap=True)
DuplicateKey(keyId, position, snap=True)
```
Key one channel at `position` in the active variant, at `value` (default: the timeline's value there, so the curve keeps its shape); a key already at that spot takes it. Or copy a key (every variant, its curve, length, cut and per-channel transitions) to `position`. Both select and return the key.

```python
BackToTimeline(names=None, glide=True)
Overridden
Touched()
```
Hand the named channels (default: all) back to the timeline, gliding when **Glide** is above 0. `Overridden` lists the channels that stopped following; `Touched()` the ones a Store would key now.

```python
TogglePlay()
StopTransport()
ToggleLoop()
SetLoop(start, end, free=False, mode='draw')
```
Play or pause (the **Play** parameter), stop and go back to the start, loop on or off (turned on with no range: 4 bars from the playhead's bar, or 4 seconds from the playhead's second), or set the loop range on the grid unless `free`: `mode` `'draw'` snaps both ends and turns Loop on, `'start'` / `'end'` snap that end, `'move'` snaps the start and keeps the length.

```python
SetUnit(unit, snap=False)
```
Switch the timeline to `'seconds'` or `'beats'`. Every key, transition length, loop and Position is converted through the tempo, so nothing moves in time. `snap=True` (to beats) moves each key to its nearest beat, unless another key in the same variant is already there. Returns False when the timeline already counts in that unit.

```python
SelectKey(keyId)
StepKey(direction)
```
Select a key and move Position onto it (in Edit mode, also pose the target). `StepKey(1)` / `StepKey(-1)` select the next or previous key of the active variant.

```python
MoveKey(keyId, position, snap=True)
SetKeyField(keyId, field, value)
SetKeyValue(keyId, variant, par, value)
```
Move a key (it keeps its marker), set one of `marker`, `offset`, `cut`, `curve`, `length`, or set one stored value. Each returns False when it cannot apply: an unknown key or field, a marker missing from the table, or a key that does not hold that variant or channel.

```python
LoadPose()
```
Pose the target with the timeline's values at Position.

```python
SetAngleChannels(pattern)
ApplyChannels()
```
What the **Angle Channels** and **Channels** parameters call. Each returns True when the channels changed.

```python
AddVariant(name, source=None)
RenameVariant(name, newName)
RemoveVariant(name)
```
Make a variant as a copy of `source` (default: the active one), rename one, or drop one with its values (a key left with no values goes; the last variant stays). Each returns False when it cannot apply.

```python
Timeline
SetTimeline(timeline)
ImportFile(path)
```
The timeline as a plain dict (a copy), replace it, or load it from a JSON file.

```python
Evaluate(position)
Channels
```
The values at a position, without writing anything, one per channel that some key holds (the `out1` channels, in the order of `Table['names']`). `Channels` lists every channel.

```python
Status
Report(message)
```
The current problem, empty when there is none. `Report` shows a message of your own on Status and the editor's status bar until the next edit.

```python
CurveNames()
MarkerNames()
```
The transition curves, and the markers the timeline can see.

```python
OpenEditor()
Refresh()
```
Open the editor window; redraw it.
