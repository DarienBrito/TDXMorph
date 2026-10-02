# Preset Inspector

**Part of the TDXMorph Toolkit**  
Copyright © 2026  
**Author:** [Darien Brito](https://www.darienbrito.com)  
**License:** [MIT License](https://opensource.org/license/mit)  
**Version:** 1.5.7

---

## Overview

`PresetInspector` is a **viewer and editor for preset data**. Attach it to a PresetManager and it shows you every stored preset as a navigable tree, with editable values. Point it at a JSON file instead and it does the same for that file.

It is a thin wrapper: the tree itself is an embedded `JSONTree` (also MIT, also shipped standalone in this repository). The Inspector's own job is choosing the source and writing edits back to it.

Use it to check what a preset actually captured, to fix a value by hand without re-storing the whole preset, and to inspect exported JSON before sharing it.

---

## Table of Contents

- [Architecture](#architecture)
- [Quick start](#quick-start)
- [Custom parameters](#custom-parameters)
- [Modes](#modes)
- [Editing and write-back](#editing-and-write-back)
- [Reading a preset](#reading-a-preset)
- [Public methods](#public-methods)

---

## Architecture

| **Attribute** | **Description** |
|---|---|
| **Class** | `extPresetInspector` |
| **Role** | Source selection and UI forwarding for an embedded JSONTree. |
| **Embeds** | `JSONTree` (MIT). |
| **License** | MIT |

The data path is `eval1` or `filein1`, then `switch1`, then the embedded `JSONTree` input. The Inspector forwards its Search, Expand All and Collapse All controls to the tree, and mirrors its `Editable` toggle onto it.

---

## Quick start

1. Drop `PresetInspector.tox` next to a PresetManager.
2. Leave `Mode` on **Preset manager**.
3. Set `Presetmanager` to your PresetManager component. It starts empty, so the Inspector shows nothing until you point it at one.

The tree fills with the stored presets. Double-click a value to edit it.

---

## Custom parameters

### Inputs page

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Mode` | Menu | `Preset manager` | The data source: an attached PresetManager, or a JSON file. |
| `Presetmanager` | OP | *(empty)* | The PresetManager to read, in Preset manager mode. |
| `Json` | File | `''` | The JSON file to read, in JSON file mode. |
| `Editable` | Toggle | `On` | Allow editing values. Mirrored onto the embedded tree. |
| `Search` | Str | `''` | Filter the tree. Forwarded to the tree's own filter and shown in its search field. |
| `Expandall` | Pulse | | Expand every branch. |
| `Collapseall` | Pulse | | Collapse every branch. |

### About page

The `Readme`, `Help`, `Support` and `Website` pulse buttons, then `Author` and `Version`, which are **read only**.

### Look page

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Skin` | Menu | `Carbon` | Colour skin: `Carbon`, `Verdigris` or `Mono`. Handed down to the embedded tree, which owns the colours. |
| `Applyskin` | Pulse | | Re-apply the current skin. |

---

## Modes

### Preset manager

Reads `Presets` from the component named in `Presetmanager` and renders it live. Store or delete a preset and the tree follows.

### JSON file

Reads the file in `Json`. The file input is bypassed while the parameter is empty, so an unset path does not raise a warning.

This mode works on any JSON, not only TDXMorph exports, which makes the Inspector a general-purpose JSON browser inside TouchDesigner.

---

## Editing and write-back

With `Editable` on, double-click a value to edit it. The edit is coerced back to the old value's type by the tree, then handed to the Inspector's `WriteBack`, which persists it to whichever source is active.

**Values only.** You can change what a parameter is set to. You cannot add, rename or delete keys from the tree.

Write-back is guarded, and a refused write is recorded rather than silently dropped. The Inspector stores the reason in `lastWriteError`:

| **Condition** | **Recorded reason** |
|---|---|
| The `Presetmanager` parameter does not resolve | `PresetManager not resolved` |
| The PresetManager has its `Lock` on | `PresetManager is locked` |
| The edited key path is not in the presets | `path not found in presets` |
| The JSON file cannot be read | `cannot read JSON file` |
| The JSON file cannot be written | `cannot write JSON file` |
| The edited key path is not in the file | `path not found in JSON file` |

A successful write clears it to an empty string and re-cooks the source, so the tree rebuilds from the committed data rather than from what you typed.

```python
op('PresetInspector').fetch('lastWriteError', '')
```

Note the PresetManager `Lock` guard: locking the manager also locks the Inspector out of it, which is what you want during a performance.

---

## Reading a preset

TDXMorph presets use the v2 schema, so a stored preset looks like this in the tree:

```
intro
└── states
    └── /project1/noise1
        ├── params
        │   ├── period      1.0     Float
        │   └── amp         0.5     Float
        ├── time            2.0
        ├── curve           Linear
        └── endmode         hold
```

The `params` entries render as **single rows** rather than nested branches, because the embedded tree is schema-aware: a dict carrying `paramName`, `value` and `type` collapses into one row showing the name, an attribute strip, the value and a type badge. A locked parameter shows an amber lock in the gutter.

Turn the tree's own `Schemaaware` toggle off if you want to see the raw nested structure.

---

## Public methods

```python
WriteBack(keys, newVal, oldVal=None)
```
Persist an edit to the active source. `keys` is the key chain from the root of the data to the leaf. Called by the embedded tree on a committed edit; you rarely need to call it yourself.

```python
ApplySkin(name=None)
```
Switch the colour skin, returning the name the embedded tree reports back. The Inspector owns no colours itself: it sets the tree's `Skin` and hands the call down.

Everything else the Inspector does is driven from its parameters, and the tree's own API is available on the embedded `JSONTree` component. See [JSONTree](JSONTree.md).

