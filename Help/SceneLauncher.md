# SceneLauncher

The root of the SceneLauncher component. Manages scenes, their preset lists and the
playback of both, driving an external PresetManager.

Class: `extSceneLauncher`. Version 4.10.1.

> **PROPRIETARY. Licensed, not sold.** A commercial component governed by the
> SceneLauncher EULA (see the `LICENSE` operator inside the component). Available
> through [Patreon](https://www.patreon.com/c/darienbrito), not from this repository.

## Core level methods

### Properties (read/write)

```python
Source = v
```
v is a preset manager

```python
Row = v
```
v is current row

```python
Col = v
```
v is current col

```python
Scene = v
```
v is current scene

```python
Target = v
```
v is current target

### Promoted

```python
ClearActions()
```
Clear all actions listed in the Scene launcher.

```python
RegisterMapTargets()
```
Tell the ControlMapper which controls are mappable, and return how many were registered. A control is mappable when it carries a `MapTarget` child. Library templates are skipped, and any widget with `Ignoreprotocols` on is left out, which is how the header chrome stays out of the map. Called on init; call it again after adding controls at runtime.

```python
SetMapMode(state)
```
Enter or leave map mode, revealing a click target on every mappable control. Re-registers first, so a control added since load still gets a target. This is what the header **Map** icon drives.

```python
SelectRow(row)
```
Set the active scene row and highlight it on the scene list. `row` is the 1-based display row. Promoted in 4.8.0 (it was `selectRow`), because it is transport feedback a host may want to drive.

#### Scripting API (4.8.0)

Seventeen methods for driving scenes and presets from a script, a MIDI/OSC handler or
another component, without going through the lists. None of them opens a dialog: a call
that cannot be honoured returns `False`, `[]` or `None`, so an automated path can never
block waiting for a click.

`GetScene` and `GetPreset` hand back **copies**. Mutating what they return does not touch
the stored model.

```python
GetScenes()
```
Scene names, in display order.

```python
GetScene(name)
```
A copy of one scene's data, or `None` if there is no such scene.

```python
RenameScene(old, new)
```
Rename a scene, keeping its row position. `False` if `old` is unknown, or `new` is blank or already taken.

```python
SetSceneField(name, field, value)
```
Set one editable field: `name`, `length`, `delay`, `target`, `curve`, `action`, `script`, `color`. `False` on an unknown scene or an unsupported field. Setting `length` or `curve` also overwrites that value inside the scene's target preset, exactly as editing the cell by hand does. A curve of `'None'` leaves the preset alone.

```python
SetSceneTarget(name, target)
```
Point a scene at a preset; `'None'` detaches it. `False` if the scene is unknown or the preset does not exist on the attached PresetManager.

```python
SetSceneCurve(name, curve)
```
Set a scene's morph curve. `'None'` makes the scene morph with its preset's own curve, and leaves that preset untouched. `False` on an unknown scene, or a curve the attached engine does not offer.

```python
DeleteScene(name)
```
Delete one scene. `False` if there is no such scene. The transport stays on the scene it was on; deleting that scene moves it to the next one.

```python
ClearScenes()
```
Delete every scene and put the transport back at the start. Returns `True` even when the list was already empty.

```python
ReorderScenes(order)
```
Reorder scenes by current 0-based row indices, where `order[newRow] = oldRow`. Refuses anything that is not a full permutation of the existing rows, so a partial list cannot silently drop scenes. The transport cue follows the row it was on.

```python
LaunchSceneByName(name)
```
Launch a scene by name. `False` if there is no such scene, so a mistyped name cannot fire the wrong one.

```python
GetPresetNames()
```
Preset names from the attached PresetManager, in order. Empty when nothing is attached.

```python
GetPreset(name)
```
A copy of one preset's stored data, or `None` if it is unknown or nothing is attached.

```python
StorePreset(name)
```
Capture the current parameter state into `name`, creating or overwriting it. `False` when nothing is attached, when the name is blank, or when the PresetManager has no parameters to capture, which includes the case where one of its stored paths no longer resolves.

```python
RenamePreset(old, new)
```
Rename a preset, keeping its position, and update every scene aiming at it. A PresetManager Target on the old name follows it.

```python
DeletePreset(name)
```
Delete a preset, and reset every scene aiming at it to `'None'`. A PresetManager Target on it falls back to the first preset.

```python
ClearPresets()
```
Delete every preset, and reset every scene target to `'None'`. The cascade is new in 4.8.0; before it, clearing left scenes pointing at names that no longer existed.

```python
ReorderPresets(order)
```
Reorder presets by current 0-based row indices. Same permutation rule as `ReorderScenes`.

```python
CreateScene(name, target='None')
```
Create a new Scene with optional target.

```python
DelayedPresetTrigger(target, length, curve)
```
Trigger a preset with a delay.

```python
DuplicateScene(name, sourceName)
```
Copy sourceName into a new scene and return the name it got. Never overwrites: a taken name gets a
numeric suffix. Returns `''` when sourceName does not exist.

```python
EnableFollowActions(enable=True)
```
Arm or disarm the follow action that runs when the current morph completes. A scene launch arms it once its morph starts. Stop, a newer launch, or any other morph starting on the PresetManager disarms it, so only the scene's own morph fires its follow action.

```python
EnableScripting(enable=True)
```
Show or hide the Script column in the scene list. It only hides the column: a stored script still runs when its scene launches.

```python
ExportPresetsJSON()
```
Export current presets including scenes to disk.

```python
GetCueActions()
```
Grab available morph curves from source.

```python
GetCurrentCellValue(cellName)
```
Grab acurrent cellName data.

```python
GetMorphCurves()
```
Grab available morph curves from source.

```python
ImportPresetsJSON()
```
Import presets including scenes from disk. The presets go to the PresetManager before any scene is replaced, so a refused file changes nothing.

```python
PerformAction(action, value=None)
```
Trigger some action from the transport menu. Value is only useful for Play/Pause action. Play/Pause leaves a running Sequence alone; every other action ends it.

```python
ReportResult(msg, title)
```
Creates a TDXMorph-formatted pop-up window with given message and title.

```python
SequentialLaunch(data)
```
This is a special method used exclusively by the "Sequence" action. See SceneLauncher/Actions to understand the logic.

```python
SetCellColor(row, col, color)
```
Re-render the scene list after a colour change. The colour itself is written to the scene by the caller.

```python
WritePresets()
```
Write the found presets into the table in this component. This gets recalled automaticall everytime the presets changed.

```python
WriteScenes()
```
Write the found scenes into the table in this component. This gets recalled automatically everytime the scenes changed.

### Private

> Renamed in 4.8.0. Six of these gained a leading underscore to mark them private, and
> `selectRow` was promoted to `SelectRow` (documented above). If you called any of them
> from your own code, update the name: the old spellings no longer exist.

```python
_assembleData()
```
Manually recreate a minimal info dictionary to be used with the lister callbacks. Was `assembleData`.

```python
_randomColor()
```
Returns a random color for the cells. Was `getRandomColor`.

```python
_launchCell(row)
```
Triggers the "Launch" cell at given row. Was `launchCell`.

```python
_disableBlending()
```
Turn blending off on the attached PresetManager. Blending and morphing are mutually exclusive there, and a scene launch always triggers a morph. No-op when nothing is attached.

```python
onAny(info)
```
Execute homonimus action. info is lister callback info object.

```python
onFirst(info)
```
Execute homonimus action. info is lister callback info object.

```python
onLast(info)
```
Execute homonimus action. info is lister callback info object.

```python
onNext(info)
```
Execute homonimus action. info is lister callback info object.

```python
onOther(info)
```
Execute homonimus action. info is lister callback info object.

```python
onPlayPause(info, value)
```
Execute homonimus action. info is lister callback info object. value is on or off.

```python
onPrevious(info)
```
Execute homonimus action. info is lister callback info object.

```python
onRepeat(info)
```
Execute homonimus action. info is lister callback info object.

```python
onSequence()
```
On sequence plays the whole scenes sequence, ignoring the actions column. This is handled in the actions callback.

```python
onStop(info)
```
Stop the morph, cancel launches still waiting out their Delay, disarm the pending follow action and end a running Sequence.

```python
_updateTimeInfo(totalDuration)
```
Updates the COMP with total duration calculated from the sum of all scenes, including delays. Was `updateTimeInfo`.

## UI level methods

### Promoted

```python
CreateAnimation()
```
Creates an animation from the set of scenes in the SceneLauncher: one key per scene, in order, with the scene's length, and a scene's Delay holds the previous scene for that long. Scenes without a valid target are skipped; with nothing to animate it is refused and creates nothing.

### Private

```python
_createAnimationCOMP(x=250, y=0, viewer=True)
```
Creates a copy of the requested item at the location of TDXMorph. Possible objects are PresetManager, PresetsGrabber and PresetsAnimator. Was `createAnimationCOMP`.
