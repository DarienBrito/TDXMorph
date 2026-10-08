# JSONTree

**Part of the TDXMorph Toolkit**  
Copyright © 2026  
**Author:** [Darien Brito](https://www.darienbrito.com)  
**License:** [MIT License](https://opensource.org/license/mit)  
**Version:** 1.5.8

---

## Overview

`JSONTree` is a reusable **collapsible tree viewer and inline editor** for JSON and for Python data structures, built on TouchDesigner's built-in `listCOMP`.

Point it at a file, a wired input or a Python object and it renders a navigable tree with search, expand and collapse, type-coloured values and editable leaves. It is the viewer embedded inside `PresetInspector`, and it works equally well on its own for any JSON you need to inspect inside TouchDesigner.

MIT licensed, with no palette or third-party content.

---

## Table of Contents

- [Architecture](#architecture)
- [Quick start](#quick-start)
- [Custom parameters](#custom-parameters)
- [Sources](#sources)
- [Schema-aware parameter rows](#schema-aware-parameter-rows)
- [Editing](#editing)
- [The toolbar](#the-toolbar)
- [Public methods](#public-methods)
- [Host integration](#host-integration)

---

## Architecture

| **Attribute** | **Description** |
|---|---|
| **Class** | `extJSONTree` |
| **Role** | Model, rendering and interaction for a collapsible JSON tree. |
| **Built on** | The native `listCOMP`, plus native text and button COMPs for the toolbar. |
| **License** | MIT |

---

## Quick start

Wire a DAT containing JSON into the component's input, or push data from Python:

```python
tree = op('JSONTree')

tree.SetData({'scene': {'name': 'intro', 'level': 0.4, 'active': True}})
# or
tree.SetJSON('{"scene": {"name": "intro"}}')
```

Both set `Source` to `Data` for you.

---

## Custom parameters

### Inputs page

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Source` | Menu | `Input` | Where the data comes from: `Input`, `File` or `Data`. |
| `File` | File | `''` | The JSON file to read when `Source` is `File`. |
| `Search` | Str | `''` | Filter the tree. Branches whose subtree matches stay visible. A saved search filters again on reopen. |
| `Expandall` | Pulse | | Expand every branch. |
| `Collapseall` | Pulse | | Collapse every branch. |
| `Editable` | Toggle | `On` | Allow inline editing of leaf values. |
| `Schemaaware` | Toggle | `On` | Render TDXMorph parameter dicts as single rows. See below. |
| `Showtoolbar` | Toggle | `On` | Show the in-panel toolbar. |

### Callbacks page

| **Parameter** | **Type** | **Description** |
|---|---|---|
| `Onedit` | Pulse | Fired by the component after a committed edit, for a host to watch. |

### About page

The `Support` (labelled Patreon) and `Website` pulse buttons, then `Author` and `Version`, which are **read only**.

### Look page

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Skin` | Menu | `Carbon` | Colour skin: `Carbon`, `Verdigris` or `Mono`. Repaints the tree and the toolbar. |
| `Applyskin` | Pulse | | Re-apply the current skin. This is the way back after hand-editing a colour. |

---

## Sources

| **`Source`** | **Behaviour** |
|---|---|
| `Input` | Read the JSON from the component's wired input DAT. The tree rebuilds automatically when that DAT changes. |
| `File` | Read from the path in the `File` parameter. The file input is bypassed while no path is set, so an empty parameter does not raise a warning. |
| `Data` | Hold data pushed in from Python with `SetData` or `SetJSON`. |

---

## Schema-aware parameter rows

With `Schemaaware` on (the default), a dict that looks like a TDXMorph parameter state, meaning it carries `paramName`, `value` and `type` keys, collapses into **one row** rather than expanding into a four-key branch.

That row shows the parameter name, an inline attribute strip, the value and a type badge. Remaining keys such as `min`, `max`, `normMin` and `normMax` render as the attribute strip and are display-only. A `locked` parameter gets an amber lock in the gutter.

This is what makes a TDXMorph preset readable: a preset with thirty tracked parameters shows thirty rows instead of a hundred and twenty nested keys.

Turn `Schemaaware` off to see the raw structure, which is useful when debugging the stored shape itself.

---

## Editing

With `Editable` on, double-click a value cell to edit it in place. The new text is **coerced back to the old value's type**, so editing a float leaves a float and a bad numeric entry is rejected rather than silently turning the value into a string. A whole number takes a typed fraction and becomes a decimal; a whole number typed over it stays a whole number.

An edit is persisted to the tree's own source when that source can accept it, meaning the `Data` object or the `File`. The component then fires the `Onedit` pulse so a host can react.

If a persist fails, the component reports it rather than announcing a success it did not achieve.

---

## The toolbar

The in-panel toolbar carries a search field and an expand/collapse pair. The search field shows a magnifier glyph as its placeholder when empty, and drives the `Search` parameter. The chevron buttons drive `Expandall` and `Collapseall`.

Hide the whole strip with `Showtoolbar` when you are embedding the tree in your own UI and want to supply the controls yourself.

---

## Public methods

```python
SetJSON(text)
SetData(obj)
```
Push raw JSON text, or a Python dict or list, into the tree. Both set `Source` to `Data`.

```python
Rebuild()
```
Re-parse the source, rebuild the node model and re-render.

```python
RebuildIfChanged()
```
Rebuild only when the source text or `Schemaaware` changed since the last rebuild. Returns True when it rebuilt. The tree uses it for its own source watching, so one edit parses once.

```python
ExpandAll()
CollapseAll()
```
Expand or collapse every branch.

```python
ExpandUnder(nodeId)
```
Expand a whole subtree at once. This is what a double-click on a branch calls.

```python
Toggle(nodeId)
```
Expand or collapse a single branch by one level.

```python
SetFilter(text)
```
Filter the tree, the same as setting the `Search` parameter.

```python
CopyCell(value)
```
Copy a cell's value to the clipboard.

```python
EditCell(row, col, val)
```
Commit an edit programmatically, with the same coercion and persistence as a manual edit.

```python
ApplySkin(name=None)
```
Switch the colour skin and redraw, returning the name that took effect. With no name it reads the `Skin` parameter, and an unknown name falls back to the theme default.

---

## Interaction reference

| **Action** | **Result** |
|---|---|
| Single-click a chevron or key | Expand or collapse that branch by one level. |
| Double-click a branch | Expand the whole subtree beneath it. |
| Double-click a value | Edit it in place, if `Editable` is on. |
| Type in the search field | Filter the tree to matching branches. |

---

## Host integration

To embed the tree in your own component, wire your JSON into it and watch the `Onedit` pulse:

```python
# in a parexec watching the embedded tree's Onedit
def onPulse(par):
	tree = par.owner
	# read back whatever the tree just changed and persist it your own way
	return
```

`PresetInspector` in this repository is a worked example: it embeds a JSONTree, owns the source selection itself, and implements `WriteBack` to push edits into an attached PresetManager or a JSON file.

