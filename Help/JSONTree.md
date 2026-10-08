# JSON Tree

A reusable collapsible JSON and data tree viewer with inline editing, built on the native
`listCOMP`. Shipped standalone and embedded inside the PresetInspector.

Version 1.5.8. MIT. Capitalized methods are promoted and are the supported API.

Full reference: [Documentation/JSONTree.md](../Documentation/JSONTree.md).

## Core level methods

### Properties

These are custom parameters rather than Python properties.

```python
Source = str
```
Where the data comes from: `Input`, `File` or `Data`.

```python
File = str
```
The JSON file to read when Source is File. The file input is bypassed while this is empty, so an unset path raises no warning.

```python
Search = str
```
Filter the tree. Branches whose subtree matches stay visible.

```python
Editable = bool
```
Allow inline editing of leaf values.

```python
Schemaaware = bool
```
Render a dict carrying paramName, value and type as a single row rather than a four-key branch.

```python
Showtoolbar = bool
```
Show the in-panel toolbar (search field plus expand and collapse).

```python
Skin = str
```
Colour skin: Carbon, Verdigris or Mono. Setting it repaints the colour parameters.

### Promoted

```python
ApplySkin(name=None)
```
Switch the colour skin and redraw. Returns the name that took effect. With no name it reads the Skin parameter; an unknown name falls back to the theme default.

```python
CollapseAll()
```
Collapse every branch.

```python
CopyCell(value)
```
Copy a cell's value to the clipboard.

```python
EditCell(row, col, val)
```
Commit a value-cell edit: coerce to the old type, persist to the source, then notify the host.

```python
ExpandAll()
```
Expand every branch.

```python
ExpandUnder(nodeId)
```
Expand a whole subtree at once. This is what a double-click on a branch calls.

```python
Rebuild()
```
Re-parse the source, rebuild the node model and re-render.

```python
RebuildIfChanged()
```
Rebuild only when the source text or Schemaaware changed since the last rebuild. Returns True when
it rebuilt.

```python
SetData(obj)
```
Push a Python dict or list into the tree. Sets Source to Data.

```python
SetFilter(text)
```
Filter the tree. Equivalent to setting the Search parameter.

```python
SetJSON(text)
```
Push raw JSON text into the tree. Sets Source to Data.

```python
Toggle(nodeId)
```
Expand or collapse a single branch by one level.

### Called by the listCOMP

Dispatched by the docked callbacks. You do not call these.

```python
HandleClick(row, col)
RenderCell(row, col, attribs)
RenderCol(col, attribs)
```

## Callbacks

```python
Onedit = pulse
```
Fired by the component after a committed edit, on the Callbacks page, for a host to watch
with a parameter execute.

## Notes

An edit is coerced back to the old value's type, so editing a float leaves a float and a bad
numeric entry is rejected rather than silently becoming a string. A whole number takes a typed
fraction and becomes a decimal.

A persist that fails is reported rather than announced as a success.

Schema-aware rows are what make a TDXMorph preset readable: a preset with thirty tracked
parameters shows thirty rows instead of a hundred and twenty nested keys. Attributes such as
min, max, normMin and normMax render as a display-only strip, and a locked parameter gets an
amber lock in the gutter.
