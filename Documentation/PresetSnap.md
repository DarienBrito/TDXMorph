# PresetSnap

**Part of the TDXMorph Toolkit**  
Copyright © 2026  
**Author:** [Darien Brito](https://www.darienbrito.com)  
**License:** [MIT License](https://opensource.org/license/mit)  
**Version:** 1.3.0

---

## Overview

`PresetSnap` is **plain store and recall for any COMP**. Drop the `.tox` into a base or container and a `Presets` page appears on that component. Storing captures the live parameter values, recalling snaps them back.

It does not interpolate at all, and that is the whole point of it. Where [PresetManager](PresetManager.md) owns a morph engine and moves between states over time, PresetSnap owns nothing but a table. If you want curves, timing and multi-track morphing, use PresetManager. If you want a snapshot you can fire from a parameter, use this.

**It has no interface of its own.** PresetSnap is a base COMP with no panel: the page it injects onto your host *is* the interface, and every action is a real parameter you can map to MIDI or OSC. The one visible piece is the parameter editor, where you choose what gets captured, and it opens in its own window on demand.

Nothing in it cooks at rest: one table and two execute operators, all event driven, no timer and no per-frame Python. With no panel, there is nothing to draw either.

---

## Table of Contents

- [Architecture](#architecture)
- [Quick start](#quick-start)
- [The model](#the-model)
- [Custom parameters](#custom-parameters)
- [Storing and recalling](#storing-and-recalling)
- [Adding operator parameters](#adding-operator-parameters)
- [What is never captured](#what-is-never-captured)
- [Removing the tool](#removing-the-tool)
- [Saving presets to a file](#saving-presets-to-a-file)
- [Public methods](#public-methods)

---

## Architecture

| **Attribute** | **Description** |
|---|---|
| **Class** | `extPresetSnap` |
| **Role** | Capture and restore a host COMP's parameter values, backed by one table. |
| **Embeds** | `ListView` (MIT), as the parameter editor. |
| **License** | MIT |

The component installs itself on drop. Its Execute operator fires on create, which is what loading a component from a file is, and calls `Attach`. That also runs on every project load, so `Attach` is idempotent by contract: it creates the page when missing, refreshes the preset menu, re-marks rows whose parameter has vanished, and never duplicates anything.

---

## Quick start

1. Drop `PresetSnap.tox` into the COMP whose parameters you want to snapshot.
2. Look at that COMP's parameters. A new `Presets` page is there.
3. Set your parameters the way you want them, type a name in **Preset name** and pulse **New Preset**.
4. Change the parameters again, then pulse **New Preset** a second time.
5. Switch `Preset` between the two entries. The values snap.

There is no configuration step. Every custom parameter on the host is tracked by default.

**Preset name** is optional. Leave it empty and the preset is named for you (`Preset 3`), which is what lets a MIDI mapped **New Preset** work without ever stopping to ask you anything. Whatever you type is consumed by the next **New Preset** or **Rename**, so it cannot be applied twice by accident.

**Edit Params** opens the parameter editor in its own window. That is where you choose which parameters are captured, and where you drag operators in to add theirs.

---

## The model

Everything is one table operator called `presets`. Rows are parameters, columns are presets.

```
param        include  mode      Intro  Drop
Speed        1        constant  0.25   1.0
Seed         1        constant  7      312
Active       1        constant  1      0
noise1.period 1       constant  4.0    0.5
Rate         1        expression
```

Every operation is a table operation. Store writes a column, recall reads one, New appends one, Delete drops one, Rename edits a header cell, reordering is moving a column, and including or excluding a parameter is flipping a cell.

There is no JSON blob, no storage dictionary and no migration code. The consequences are worth stating plainly: the data is visible in the network, editable by hand, saved with the host, and diffable if you file-sync the table.

A row token is the parameter name for a host parameter (`Speed`), or `<operator>.<parameter>` for a parameter on an operator inside the host (`noise1.period`).

The `mode` cell records the parameter's mode as TouchDesigner reports it, lowercased, so `constant` rather than `const`. Toggles serialize as `1` and `0` rather than `True` and `False`, because the table is meant to be hand edited and `1`/`0` is what a parameter dialog shows. Recall still accepts `true`, `false`, `on` and `off` coming back the other way.

---

## Custom parameters

### Host page (injected)

These seven appear on the COMP you dropped the component into, not on the component itself. **They are the whole interface.** Being real parameters, they are MIDI and OSC mappable and work with no window open anywhere, which is the reason for injecting them at all.

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Snappreset` | Menu | The current preset. **Changing it recalls**, which is why there is no separate Recall button. |
| `Snapname` | Str | A name for the next **New Preset** or **Rename**. Optional: empty means the preset is auto named. |
| `Snapnew` | Pulse | Append a new preset and capture into it, using `Snapname` if you typed one. |
| `Snapstore` | Pulse | **Overwrite** the current preset with the live values. With no preset selected, creates one. |
| `Snaprename` | Pulse | Rename the current preset to `Snapname`. |
| `Snapdelete` | Pulse | Delete the current preset, after a confirmation that names it. |
| `Snapedit` | Pulse | Open the parameter editor in its own window. |

The order is the lifecycle of a preset: make one, overwrite it, rename it, delete it. **Store comes after New Preset because Store is the destructive one of the pair.** It replaces what the selected preset captured, with no confirmation and no undo, so the button that makes a preset reads before the button that overwrites one.

`Snapnew` and `Snaprename` both **consume** `Snapname`: they read it, then clear it. A name you typed for one preset can never silently land on the next.

A text field rather than a popup is a deliberate choice. TouchDesigner's built in dialog returns a button, not typed text, and the only text prompt in the palette would cost this component its dependency free MIT status. A parameter also keeps the controller path working: a mapped **New Preset** with an empty field names the preset itself and never blocks, where a popup would interrupt a performance. Delete is the exception and does ask, because it destroys something.

The names are deliberately namespaced. Custom parameter names must be unique across every page on a host, and `Store` is a plausible name for a host to already own. Each one carries its readable text as its label, so the page reads `Preset`, `Preset name`, `New Preset`, `Store`, `Rename`, `Delete`, `Edit Params`.

### PresetSnap page

On the component itself.

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Injectpage` | Toggle | `On` | Inject the `Presets` page onto the host. Off removes it and leaves the host untouched. |
| `Rescan` | Pulse | | Re-read the host's parameters, adding rows for any that are new. |
| `Attachpage` | Pulse | | Re-run the install by hand. |
| `Removepage` | Pulse | | Remove the injected page from the host. |

If the host already has a page called `Presets`, the tool adds its parameters to that page. Your own parameters on it are captured like any others, and removing the page takes off only the tool's parameters.

### About page

The `Readme`, `Help`, `Support` and `Website` pulse buttons, then `Author` and `Version`, which are **read only**.

---

## Storing and recalling

A store walks every row, reads the live value of each included parameter, and writes it into the preset's column. A recall walks the same rows and writes the stored value back.

Four things are skipped on recall, and the tally is returned so a UI can report it:

| **Skipped** | **Why** |
|---|---|
| The row is not included | You turned it off in the editor. |
| The parameter no longer resolves | It was renamed or deleted. The row is marked `missing` (shown as `gone`) and kept, still included. |
| The parameter is not in constant mode | See below. |
| The cell is empty | Nothing was ever captured for this preset. |

**A recall never writes a parameter that is carrying an expression, an export or a bind.** Setting a value in TouchDesigner also forces that parameter into constant mode, so a naive recall would silently convert your expression into a frozen number, with no error and no undo. The mode is re-read live at recall time rather than trusted from the table, which means a row starts working again by itself the moment you remove the expression. Such a row keeps its include flag rather than being switched off.

The editor shows the mode next to every parameter, so you can see at a glance which rows a recall will actually write.

A vanished parameter is marked, never deleted and never switched off. It may come back, and its stored values are your data: once it returns, the next rescan picks it up and a recall writes it again.

Picking a preset in the host menu reports the result on the status bar, for example `A: set 5, 2 kept, 1 failed, 1 gone`: values written, rows kept because the parameter is not constant, values that would not apply, and parameters that are gone.

A string menu takes a typed value that is not one of its items, and a recall writes it back as typed. A regular menu only accepts its own items.

---

## Adding operator parameters

The host's own custom parameters are tracked automatically. To capture parameters on operators inside the host, drag the operator onto the parameter editor.

By default that adds the parameters that are in constant mode **and** differ from their default, which is the useful subset: what you already touched is what you care about. Rows are namespaced by operator name, so a dropped `noise1` produces `noise1.period` and friends.

An operator outside the host is refused. Its row would resolve today and dangle the moment the host is copied somewhere else.

---

## What is never captured

- **The parameters this component injects.** Capturing the preset selector would make a recall rewrite it mid-recall and re-enter.
- **Pulses and momentaries.** They carry no state.
- **A sequence's block count.** The values inside each block are captured, one row per block parameter (a Constant CHOP stores `const0value`, `const1value` and so on). A recall onto fewer blocks reports the extra rows as gone.
- **Anything inside the PresetSnap component itself.**

If the host is a clone, whether its Clone parameter is a path or an expression, the tool says so on the status bar and declines to inject rather than writing a page that clone syncing would overwrite. Add PresetSnap to the clone master instead.

---

## Removing the tool

Deleting the component does **not** remove the page it injected. An Execute operator has no destroy event, and the operator in question lives inside the thing being deleted.

Pulse **Remove Page** before deleting the component, or turn `Injectpage` off, which does the same thing. Orphaned parameters left behind are inert, and you can also remove them by hand from the host's parameter dialog.

---

## Saving presets to a file

```python
op('PresetSnap').ToJSON()
op('PresetSnap').FromJSON(text)
```

`ToJSON` writes PresetSnap's own format: each preset maps a row token straight to its stored value, and the file is stamped as a PresetSnap file. `FromJSON` reads it back into any PresetSnap.

It is not the [PresetManager](PresetManager.md) format, and the two do not convert. PresetManager, ParameterMorpher and SceneLauncher refuse a PresetSnap file and leave their own presets untouched, and `FromJSON` refuses a file written by any of them, names the tool that wrote it on the status bar, and changes nothing.

---

## Public methods

```python
Store(name)
```
Write the live value of every included, constant mode parameter into the named preset. Returns a dict with `stored`, `skipped` and `error`.

```python
Recall(name)
```
Write one preset's column back onto the host. Returns the tally described in [Storing and recalling](#storing-and-recalling).

```python
NewPreset(name='')
```
Append a preset and capture into it immediately. Returns the name used. With no name given it auto names, so `Preset 3`, and you rename it in the editor afterwards. A name that is taken, or that is one of the table's own column names (`param`, `include`, `mode`), gets a number: `param 2`.

```python
Delete(name)
```
Delete a preset column. Refuses to delete a header column.

```python
Rename(old, new)
```
Rename a preset. The new name is made unique against the existing ones and the table's own column names rather than colliding.

```python
PresetNames()
```
Every preset name, in table order.

```python
CurrentPreset()
```
The name selected in the host `Preset` menu, or an empty string.

```python
Rescan()
```
Add rows for host parameters not yet listed, refresh every row's mode, and mark rows whose parameter no longer resolves. Returns `{'added': N, 'missing': N}`.

```python
AddOp(opOrPath, pars=None)
```
Add rows for an operator inside the host. With no explicit `pars`, adds the parameters in constant mode that differ from their default. Returns how many rows were added.

```python
Include(token, on)
```
Turn one row's capture flag on or off, by row token.

```python
Attach()
```
Create the host page if missing and resync everything. Idempotent, and called for you on drop and on every project load.

```python
RemovePage()
```
Remove the injected parameters from the host, and the page once nothing else is on it. Returns False when there was no page.

```python
OpenEditor()
```
Rescan, refresh the list and open the parameter editor window. What the `Snapedit` pulse calls.

```python
Host()
```
The COMP this component was dropped into, or None at the project root.

```python
ToJSON()
FromJSON(text)
```
Export and import in PresetSnap's own format (see [Saving presets to a file](#saving-presets-to-a-file)). `FromJSON` returns how many presets were read, and uniquifies any name that already exists or is one of the table's column names.

