# List View

A reusable flat-columnar list widget built on the native `listCOMP`. Shipped standalone and
used internally by the PresetManager paths editor.

Version 1.0.9. MIT. Capitalized methods are promoted and are the supported API.

Full reference: [Documentation/ListView.md](../Documentation/ListView.md).

## Core level methods

### Properties

These are custom parameters rather than Python properties.

```python
Host = COMP
```
The component owning the data model. Callbacks reach it via `lister.par.Host.eval()`.

```python
Callbacks = DAT
```
The behaviour module for this instance. Resolves relative to the lister's parent, so a sibling name works.

```python
Editable = bool
```
Master switch for inline editing. A column must also set `editable`.

```python
Header = bool
```
Show the column header row.

```python
Emptyheader = bool
```
Keep the header visible when there is no data yet, instead of rendering fully empty.

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
Build()
```
Re-read the column spec from the host, then refresh the rows.

```python
EditCell(row, col, val)
```
Commit an inline edit. Value cells are coerced to the old type (a whole number takes a fraction as a
decimal), then the host is notified.

```python
Refresh()
```
Re-read row data from the host and re-render. A selected row with a `_key` stays selected wherever it
moves, and the selection clears when that row is gone.

```python
Reorder(order)
```
Apply a new row order and notify the host.

```python
ReorderDict(d, order)
```
Static helper: reorder an ordered dependable dict in place by a list of source indices. For use inside a ListViewReorder hook.

```python
Rows()
```
The current row data.

```python
SelectedRow()
```
The selected row's data, or None when nothing is selected.

```python
SetActiveRow(dataIndex)
```
Select and highlight a row by 0-based data index and re-render. Pass -1 to clear.

```python
SetColumnAlign(col, align)
```
Set a column's text alignment (left, center or right) and persist it.

```python
SetColumnLabel(col, label)
```
Set a column's header label and persist it.

```python
SetColumns(spec)
```
Push a column spec directly, for host-less or programmatic use.

```python
SetRows(rows)
```
Push row data directly, for host-less or programmatic use.

### Called by the listCOMP

These are dispatched by the docked callbacks and the right-click watcher. You do not call
them, but they are promoted because the callbacks reach them from outside.

```python
HandleClick(row, col)
HandleRightClick()
HandleDragMove(startRow, endRow)
HandleSelectEnd(startRow, endRow)
HandleDrop(source)
SetRollover(row, col)
RenderCell(row, col, attribs)
RenderCol(col, attribs)
RenderRow(row, attribs)
```

## The Callbacks contract

Point the `Callbacks` parameter at a DAT whose module defines any of these module-level
functions. All are optional. Each takes the ListView COMP as its first argument.

```python
ListViewColumns(lister)                  # -> column spec list
ListViewRows(lister)                     # -> list of row dicts
ListViewAction(lister, name, row)        # action / raction click
ListViewActivate(lister, row)            # double-click a row
ListViewEdit(lister, row, key, new, old) # committed inline edit
ListViewReorder(lister, order)           # new order, as data indices
ListViewContext(lister)                  # right-click empty space
ListViewHeaderMenu(lister, col)          # right-click a header (overrides the built-in menu)
ListViewSelect(lister, row)              # row pressed
ListViewDrop(lister, source)             # a panel drop landed
```

Dispatch order is the Callbacks DAT module, then the `Host` component's promoted method,
then a lean built-in default.

## Column spec

One dict per column:

```
{key, label, width, stretch, align, kind, editable, action, raction,
 glyph, color, iconfont, iconsize, help}

kind  = text | value | icon | badge | swatch | index    (default text)
align = left | center | right                           (default left)
```
