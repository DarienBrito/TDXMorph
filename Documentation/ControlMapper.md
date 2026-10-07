# ControlMapper

**Part of the TDXMorph Toolkit**  
Copyright (c) 2026  
**Author:** [Darien Brito](https://www.darienbrito.com)  
**License:** [MIT License](https://opensource.org/license/mit)  
**Version:** 1.1.0

---

## Overview

`ControlMapper` is the **MIDI and OSC mapping service**. One instance serves both protocols from a single list of mappings, and a mapping targets an **operator plus a parameter name**, so any custom parameter can be driven rather than only a widget the service already knows about.

It ships from this repository as a standalone component, and it also travels embedded inside ParameterMorpher and SceneLauncher, where it replaced the per-widget learn overlays and the separate `Mapper` and `MapperSatellite` classes.

The map itself is a plain DAT table, not component storage. You can read it, diff it, edit it by hand and export it.

MIT licensed, with no palette or third-party content.

---

## Table of Contents

- [Architecture](#architecture)
- [Quick start](#quick-start)
- [Custom parameters](#custom-parameters)
- [The mapping table](#the-mapping-table)
- [Learning a mapping](#learning-a-mapping)
- [Ranges and takeover](#ranges-and-takeover)
- [Dead mappings](#dead-mappings)
- [The editor](#the-editor)
- [Callbacks](#callbacks)
- [Public methods](#public-methods)
- [Host integration](#host-integration)

---

## Architecture

| **Attribute** | **Description** |
|---|---|
| **Class** | `extControlMapper` |
| **Role** | One mapping service for both protocols: learn, store, route. |
| **Model** | The `table_map` DAT. Component storage is empty by design. |
| **Built on** | Native CHOPs for input, one execute DAT for learn and route, one click router. |
| **License** | MIT |

MIDI and OSC arrive as two `selectCHOP`s, each normalised to 0..1 by its own `mathCHOP`, then merged into one channel namespace. A single execute DAT branches learn against route, so there is no second engine and no per-protocol inspector.

Routing runs off a compiled dictionary of resolved `Par` objects, rebuilt only when the table changes. One message fanned out to 200 targets costs about 0.36 ms.

A mappable control is any COMP carrying a `MapTarget` child. That child is one operator with no logic and no extension: it exists only to swallow the click while map mode is on.

---

## Quick start

Drop the component in, point it at your MIDI or OSC input, and let it find your controls:

```python
mapper = op('ControlMapper')

mapper.par.Midi = op('midiin1')
mapper.ScanTargets()
mapper.SetMapMode(True)
```

With map mode on, every registered target reveals itself. Click one to arm it, then move a control on your MIDI device or send an OSC message: the next channel that arrives binds to it.

Leave map mode and the mappings route:

```python
mapper.SetMapMode(False)
```

---

## Custom parameters

### Settings page

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Midi` | CHOP | `''` | The MIDI input CHOP. |
| `Midifromrange1` | Float | `0.0` | Low end of the incoming MIDI range. |
| `Midifromrange2` | Float | `127.0` | High end of the incoming MIDI range. |
| `Osc` | CHOP | `''` | The OSC input CHOP. |
| `Oscfromrange1` | Float | `0.0` | Low end of the incoming OSC range. |
| `Oscfromrange2` | Float | `1.0` | High end of the incoming OSC range. |
| `Maproot` | OP | `''` | The root targets are keyed against. Left empty it falls back to the component's own parent, which is what an embedded instance wants. |
| `Targettag` | Str | `mapTarget` | The tag `ScanTargets` looks for. |

Both ranges normalise their protocol to 0..1 before anything downstream sees it, which is why one routing path serves both.

### Controls page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Mapmode` | Toggle | Enter or leave map mode. |
| `Openeditor` | Pulse | Open the mapping editor. |
| `Scantargets` | Pulse | Find targets by tag under `Maproot` and register them. |
| `Prunedead` | Pulse | Delete every mapping whose target no longer resolves. |
| `Clearmappings` | Pulse | Drop every mapping. Asks first. |

### Look page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Skin` | Menu | Colour skin: `Carbon`, `Verdigris` or `Mono`, default `Carbon`. Forwarded to the mapping editor's list, which owns the colours. |
| `Applyskin` | Pulse | Re-apply the current skin. This is the way back after hand-editing a colour. |

### Info page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Nummappings` | Int | How many mappings are stored. Read only, tracks the table. |

### Callbacks page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Callbacks` | DAT | Your callbacks DAT. See [Callbacks](#callbacks). |

### About page

The `Readme`, `Help`, `Support` and `Website` pulse buttons, then `Author` and `Version`, which are **read only**.

---

## The mapping table

The model is the `table_map` DAT, one row per mapping:

| **Column** | **What it holds** |
|---|---|
| `chan` | The incoming channel name, for example `ch1ctrl9`. |
| `target` | The target operator, keyed **relative** to `Maproot`. |
| `par` | The parameter name on that operator. Defaults to `Value`. |
| `mode` | How the value is applied. Always `absolute` in this version. |
| `lo` | Low end of the target range. |
| `hi` | High end of the target range. |
| `takeover` | `jump` or `pickup`. |

Targets are stored relative on purpose. An absolute path is what once leaked a project name into a shipped component.

---

## Learning a mapping

Learning is a three step cycle: enter map mode, arm a target, send a channel.

```python
mapper.SetMapMode(True)
mapper.Arm(op('mySlider'), 'Value')
```

The next channel that arrives calls `Map` for you and disarms. The teaching message is consumed by the learn, so it does not also route.

To re-key an existing mapping onto a different control, arm the row instead of the widget:

```python
ArmRow(3)
```

The row only takes the new channel. Its range, takeover and place in the list stay as they were. Learning a control that is already mapped, by clicking it in map mode, re-keys its row the same way.

Arming the same target twice disarms it. `Cancel` disarms without changing the map.

While something is armed the service blinks it, driven by one LFO and one execute DAT for the whole host rather than a blink chain inside every control.

A parameter **driven by an expression or an export** cannot be mapped, and neither can one bound to such a parameter. A mapping writes a value, and writing one would replace the expression for good. `Arm` refuses it, and clicking it in map mode opens a short note naming the driven parameter instead of arming.

---

## Ranges and takeover

Every mapping owns its own target range, defaulted from the parameter itself at learn time:

```python
SetMappingRange(1, 0.2, 0.8)
```

A learned **menu** parameter spans `0` to `N`, not `0` to `N-1`. TouchDesigner floors a menu value to an index, so a `0..N-1` range leaves the last item reachable from a single raw MIDI value while every other item gets eight or nine of them. `Route` clamps the top, so `N` is safe.

Takeover decides what happens when the control and the parameter disagree:

| **Mode** | **Behaviour** |
|---|---|
| `jump` | Write immediately. The parameter snaps to wherever the control already sits. |
| `pickup` | Write nothing until the incoming control crosses the parameter's current value, then take over. A parameter sitting outside the mapping's range is picked up when the control reaches that end of its travel, and steps to the end of the range. A menu is picked up when the control reaches the selected item. |

Pickup re-arms: when something else moves the parameter after the control took over (a preset recall, a morph, a hand edit), the control waits again until it reaches the new value. A range wider than the parameter's own clamp does not count as a move.

`jump` is the default and is the older behaviour. `pickup` is what kills the value jump when a fader is not where the patch left the parameter.

```python
SetMappingTakeover(1, 'pickup')
ToggleMappingTakeover(1)
```

---

## Dead mappings

A mapping whose target or parameter no longer resolves is **kept and reported**, never silently skipped:

```python
DeadRows()
PruneDead()
```

The editor shows a dead target as `(missing)`. A renamed control therefore reads as broken instead of quietly doing nothing, which is what the previous engine did on every message.

A parameter that becomes driven after it was mapped is **skipped** by `Route`, never overwritten, and reported by `DrivenRows()`. The editor marks it `(driven)`. `PruneDead` leaves it alone, because the mapping still resolves and works again as soon as the parameter is constant.

---

## The editor

`Openeditor` opens the mapping list. It is a `ListView` bound **directly to `table_map`**, so there is no parallel row model to fall out of step with the map.

Nine columns: `#`, MIDI/OSC, mapped to, parameter, min, max, takeover, learn, delete.

| **Action** | **Result** |
|---|---|
| Edit the channel, min or max cell | Rewrites that mapping. |
| Click `takeover` | Flips the row between `jump` and `pickup`. |
| Click `learn` | Re-arms that row, so the next channel re-keys it. |
| Click the delete cell | Removes that mapping. |
| Drag a row | Reorders the table. |
| Right-click empty space | Prune dead, clear all, or refresh. |

The list refreshes itself on every change, so nothing needs an explicit refresh, and the service still runs headless when no editor is present.

---

## Callbacks

Point the `Callbacks` parameter at a DAT to hear about mapping activity:

```python
def onMap(info):
	# info: {'chan', 'target', 'par'}
	return

def onUnmap(info):
	# info: {'chan', 'target', 'par'}
	return

def onRoute(info):
	# info: {'chan', 'value', 'targets'}
	return
```

`onRoute` fires per routed message and reports how many targets were written, so treat it as a monitor rather than a place to do work.

---

## Public methods

### Properties

```python
Mappings
NumMappings
MapMode
```
Every mapping as a dict in table order, how many there are, and whether map mode is on.

### Map mode and learning

```python
SetMapMode(state)
```
Enter or leave map mode. Reveals every registered target and cancels any pending arm.

```python
Arm(widget, parName='Value')
```
Arm `widget.par[parName]` so the next incoming channel binds to it. Returns False, and arms nothing, when the parameter is driven by an expression or an export.

```python
OnTargetClick(target)
```
Handle a click on a `MapTarget`. Arms its widget, or disarms it if it was already armed. On a driven parameter it explains why instead of arming, and cancels any control that was armed before, so the next knob cannot land on it.

```python
Map(chan)
```
Bind the armed target to `chan`. Consumes the arm. Returns True when a row was written, and False when nothing is armed or the armed parameter has since become driven.

```python
Unmap(chan, target, parName='')
```
Remove mappings matching channel and target, and parameter name when given. Returns how many rows were removed. All three arguments are strings, not operators.

```python
Cancel()
IsArmed()
ArmedTarget()
```
Disarm, ask whether anything is armed, and read back the armed widget or None.

```python
ArmRow(rowIndex)
ArmedRowIndex()
```
Arm an existing mapping for re-learn by 1-based table row, and read back which row is armed, or -1. A row whose parameter is driven is refused with the same note as a click.

### Editing mappings

```python
SetMappingRange(rowIndex, lo, hi)
SetMappingTakeover(rowIndex, mode)
ToggleMappingTakeover(rowIndex)
SetMappingChannel(rowIndex, chan)
DeleteMappingRow(rowIndex)
ReorderMappings(order)
```
Row edits, all by 1-based table row except `ReorderMappings`, which takes 0-based data indices and is what backs drag-reorder in the editor. `ToggleMappingTakeover` returns the new value, or an empty string on failure.

### Clearing

```python
ClearMappings()
```
Drop every mapping. This is the model call and does **not** ask first.

```python
ConfirmClearMappings()
```
Ask before dropping every mapping. Returns True when the map was cleared outright, False when a confirmation is pending. **Every user-facing path should call this one**, and both the `Clearmappings` pulse and the editor's clear-all already do. With no dialog host available it clears outright rather than leaving a button inert.

### Dead mappings

```python
DeadRows()
PruneDead()
```
The 1-based indices of mappings that no longer resolve, and a delete of all of them. `PruneDead` returns how many went.

```python
DrivenRows()
```
The 1-based indices of mappings whose parameter is driven by an expression or an export, directly or through a bind. `Route` skips them and `PruneDead` does not remove them.

### Registration

```python
RegisterTargets(targets)
```
Declare which `MapTarget` operators exist. Hosts that build controls at runtime call this on create and destroy. Returns how many were registered.

```python
ScanTargets()
```
Find targets by tag under `Maproot` and register them. Type filtered on purpose, since an unfiltered search on a large host is expensive.

### Routing

```python
Route(chan, val)
```
Send one normalised channel value, in the range 0 to 1, to every target bound to it. Returns how many targets were written. A driven parameter is skipped, never overwritten.

```python
Recompile()
```
Force a rebuild of the routing dictionary. The editor calls this after a table edit.

### UI

```python
OpenEditor()
OnBlink(val)
```
Open the mapping editor, and the blink driver for the armed target. One execute operator serves the whole host.

```python
ApplySkin(name=None)
```
Switch the colour skin, returning the name that took effect. The mapping list is a ListView clone and owns the colours, so the service only forwards. It returns an empty string when no editor is present, because the service also runs headless.

---

## Host integration

A host that builds its controls at runtime owns the registration policy, and the service stays out of it. Declare your targets rather than letting the service guess:

```python
RegisterTargets([c.op('MapTarget') for c in myControls])
```

`ScanTargets` remains the convenience path for a simple host whose controls all carry the tag and sit under one root.

Three things are worth knowing when embedding it:

- **Leave `Maproot` empty** when the service sits at the top level of your component. It falls back to its own parent, which is exactly the host root, and nothing has to be rewritten when the project moves.
- **Drive `Midi` and `Osc` from your own host parameters by expression**, so the component still ships unwired.
- **The click router binds on entry to map mode**, not at registration. Outside map mode every target is hidden and clicks early-return, so the binding would be dead weight, and binding it early can reach a panel that has never been drawn.

Both ParameterMorpher and SceneLauncher in this toolkit are worked examples.

