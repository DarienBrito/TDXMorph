# PresetSnap

Plain store and recall for the COMP this component is dropped into. No morphing, no curves, no
engine. Drop the `.tox` into a base or container and a `Presets` page appears on that COMP.

**Headless.** The component is a base COMP with no panel: the seven parameters it injects onto the
host are the interface, and the parameter editor opens in its own window.

Class: `extPresetSnap`. Version 1.2.1. MIT licensed, and shipped standalone in this repository.

Full guide: [PresetSnap](../Documentation/PresetSnap.md).

The whole data layer is one table operator, `presets`. Rows are parameters, columns are presets,
and every operation is a table operation. A row token is the parameter name for a host parameter,
or `<operator>.<parameter>` for a parameter on an operator inside the host.

## Presets

```python
Store(name)
```
Write the live value of every included, constant mode parameter into the named preset. Returns
`{'stored': N, 'skipped': N, 'error': ''}`.

```python
Recall(name)
```
Write one preset's column back onto the host. Returns a tally of what was written and what was
skipped, by reason: `written`, `skipped_excluded`, `skipped_missing`, `skipped_mode`,
`skipped_empty`, `failed`.

```python
NewPreset(name='')
```
Append a preset and capture into it. Returns the name used. With no name it auto names, so
`Preset 3`. A taken name, or one of the header column names (`param`, `include`, `mode`), gets a
number: `param 2`.

```python
Delete(name)
```
Delete a preset column. Refuses to delete one of the three header columns.

```python
Rename(old, new)
```
Rename a preset. The new name is uniquified against the presets and the header column names rather
than allowed to collide.

```python
PresetNames()
```
Every preset name, in table order.

```python
CurrentPreset()
```
The name selected in the host `Preset` menu, or an empty string.

## What is tracked

```python
Rescan()
```
Add rows for host parameters not yet listed, refresh every row's mode, and mark rows whose
parameter no longer resolves. Returns `{'added': N, 'missing': N}`. A vanished parameter is marked
and kept, never deleted or switched off: it may come back, and its stored values are the user's
data.

```python
AddOp(opOrPath, pars=None)
```
Add rows for an operator inside the host. With no explicit `pars`, adds the parameters that are in
constant mode and differ from their default. Returns how many rows were added. An operator outside
the host is refused, because its row would dangle the moment the host is copied elsewhere.

```python
Include(token, on)
```
Turn one row's capture flag on or off, by row token.

## The host page

```python
Attach()
```
Create the `Presets` page on the host if missing, then resync the menu and the rows. Idempotent by
contract: it is called on drop and on every project load. Declines when the host is a clone, by a
path or an expression, since clone syncing would overwrite the page, and says so on the status bar.

```python
RemovePage()
```
Remove the injected parameters, and the page once nothing else is on it. Returns False when there
was no page. This is a real action rather than
something that happens automatically, because an Execute operator has no destroy event and it sits
inside the thing being deleted.

```python
Host()
```
The COMP this component was dropped into, or None at the project root.

## Interchange

```python
ToJSON()
FromJSON(text)
```
Export and import in PresetSnap's own format: each preset maps a row token to its stored value, and
the file is stamped as a PresetSnap file. It is not the PresetManager format; PresetManager,
ParameterMorpher and SceneLauncher refuse it and change nothing, and `FromJSON` refuses their files
the same way. `FromJSON` returns how many presets were read, 0 on a refusal, and uniquifies any name
that already exists or is a header column name.

## Editor

```python
OpenEditor()
```
Rescan, refresh the list and open the parameter editor window. What the `Snapedit` pulse calls.

```python
RefreshEditor()
```
Rebuild the editor's rows from the model. Called for you after anything that changes them, and
once on every project load, since the row table is not saved state.

## The injected parameters

`Snappreset` (menu, changing it recalls) · `Snapname` (a name for the next New or Rename, optional)
· `Snapnew` · `Snapstore` · `Snaprename` · `Snapdelete` · `Snapedit`. That order is the lifecycle of
a preset, and `Snapstore` follows `Snapnew` because Store is the destructive one: it replaces the
selected preset's captured values with no confirmation and no undo.

`Snapnew` and `Snaprename` consume `Snapname`, so a typed name applies once and is then cleared.
An empty field means the preset is auto named, which is what keeps a mapped `Snapnew` from ever
blocking. `Snapdelete` asks for confirmation first, and names the preset in the question.

## Recall safety

A recall never writes a parameter that is carrying an expression, an export or a bind. Assigning a
value in TouchDesigner also forces the parameter into constant mode, so a naive recall would
silently freeze the expression into a number with no error and no undo.

The mode is re-read live at recall time rather than trusted from the table, so a row resumes
working by itself once the expression is removed, and such a row keeps its include flag rather than
being switched off.
