# Paths

The database of operators the PresetManager tracks, and their per-path settings. Lives at
`PresetManager/Paths`.

Nodes are marked with the manager's `Trackingtag` value (`TDXMorphPath` by default), which is
how a node that has been moved is found again.

Capitalized methods are promoted and are the supported API.

Full reference: [Documentation/PresetManager.md](../Documentation/PresetManager.md#tracked-paths).

## Core level methods

### Properties

```python
TrackingTag = str
```
Get/set the tag written onto tracked nodes.

### Promoted

```python
AutoUpdatePaths()
```
Reconcile nodes that have moved by scanning for the tracking tag. Returns the path changes applied, which the PresetManager feeds to UpdatePaths to re-key the presets. A copy of a node that is still in place is ignored. When several nodes claim one moved path, that path is left unchanged and they are named.

```python
Clear(storedSettings=True, overwriteWarning=False, prune=False)
```
Remove all entries, cleaning up every tracked node's TDXMorph data. With `overwriteWarning=False` (the default) it asks first in a non-blocking dialog and clears on the click, so the call returns before anything is cleared; the dialog works like `ConfirmRemove`. `overwriteWarning=True` clears at once. `prune=True` also removes the paths' stored values from every preset.

```python
ConfirmRemove(paths, question, title, action)
```
Ask before removing `paths`, then call `action(prune)`. When no preset stores them, the dialog is Proceed / Cancel. When presets store them, it names how many and offers **Keep** (leave the presets as they are; Enter), **Remove** (drop those values from every preset) or **Cancel** (Esc). `prune` is True only for Remove.

```python
Create(path, addTrackingTag=True, settings=None)
```
Register a path: tag the node, store its tracking path and record its settings.

```python
Delete(path, storedSettings=True, ignoreWarning=False, prune=False)
```
Remove a path from the database and strip its TDXMorph tag and storage, without asking. `prune=True` also removes its stored values from every preset; otherwise the presets keep them.

```python
GetItem(path, item)
```
Read one setting for a stored path, for example its time or curve.

```python
GetNumPaths()
```
How many paths are tracked.

```python
GetPathsKeys()
```
Every tracked path.

```python
Inject(path, data)
```
Insert or overwrite a raw settings block for a path, and tag its node for tracking.

```python
IsStoredPath(v)
```
Whether the given path is tracked.

```python
NewDrop(source, name)
```
Create a new entry from a drag-and-drop of an operator onto the editor.

```python
OpenUI()
```
Open the paths editor.

```python
OverwriteAllItems(item, val)
```
Set one item to a value on every stored path.

```python
OverwriteItem(path, item, val)
```
Set one item on a single stored path.

```python
PresetsUsing(paths)
```
Names of the presets that store values for any of `paths`.

```python
PruneFromPresets(paths)
```
Remove the stored values of `paths` from every preset. Returns how many presets changed.

```python
RegisterTrackedNodes()
```
Tag every node the database names, so it can be found again after a move. Returns the paths that point at no operator.

```python
Replace(oldPath, newPath)
```
Move an entry, its tag and its tracking path from one path to another. Returns False and changes nothing when `newPath` is not an operator or is already tracked.

```python
ReportResult(msg, title)
```
Launches a TDXMorph-formatted pop up window with the given message and title.

```python
StandardPopDialog(text, title, buttons, callback, details=None, textEntry=False, escButton=2)
StandardPopMenu(info, items, callback)
```
Open a standard pop-up dialog or menu wired to a callback. `escButton` is the 1-based button Esc picks. Guarded on TouchDesigner's own `op.TDResources`.

```python
Update()
```
Rebuild the UI reference table and the editor from the stored paths.

### Private

```python
getChangedPaths()
```
Scan the search scope for tracked nodes that have moved, ignoring copies of a node still in place.

## The editor

The paths editor is an owned MIT [ListView](ListView.md) instance, not a palette Lister.

| Action | Result |
|---|---|
| Drag an operator onto the editor | Register it as a tracked path. |
| Double-click a row | Open that node in the View pane. |
| Click the View column | Open or reuse one floating network pane focused on the node. |
| Click the delete column | Remove the path, asking first as `ConfirmRemove` does. |
| Drag a row | Reorder the paths. |
| Right-click a column header | Rename or realign the column. |
| Right-click empty space | The general Update and Clear menu. |

The per-path `Time`, `Curve`, `a`, `b` and `n` columns appear only when the manager's
**Multi-Track Engine** toggle is on.
