# ElementsContainer 

> **PROPRIETARY. Licensed, not sold.** Part of the commercial ParameterMorpher
> component, available through [Patreon](https://www.patreon.com/c/darienbrito), not from
> this repository.

## Core-level methods

### Properties

```python
Name = str
```
Get/set container name

### Promoted

```python
AddBindingReference(source, bindedData, elementPath, customName=None)
```
Stores useful references from all created bindings, so that later on we can use our presets. It stores this properties in a unique name that I call "cross-check name", which is used first to see if a parameter already exists in TDXMorph. This is there because otherwise it would be possible to create two elements for the exact same parameter. Different nodes could also have the same paramter name, so we need the whole path as a key to discern from where we are grabbing data.

```python
AddScript()
```
Scripts are a special type of elements that execute on the given snap action.

```python
ChangePresetsNum(newVal)
```
Sets the number of presets in the Elements container to newVal.

```python
ClaimElementShape(settings, curve=None)
```
Record that an element's `MorphSettings` Curve a/b/n hold the shape for `curve` (default: its current curve). Until then, an element whose curve changed in this frame stores and morphs with that curve's default shape.

```python
ClearParameters()
```
Deletes all elements in the ElementsContainer, after a confirmation, and removes their stored values from every preset. The confirmation says how many presets that affects.

```python
CreateElement(source, parameter, dataSource='parameter', customName=None)
```
Handles UI creation from a stored preset or from a grabbed parameter. In here operations for correct UI resource allocation take place. Notice that we recreate the original data type for each UI element, to make the most out of the limitations	with cloning, which do not allow for a full Object Oriented approach.

```python
Delete()
```
Deletes the ElementsContainer.

```python
ExportPresetsJSON()
```
Export all stored presets to a JSON file in disk. Notice that this is a special method of SlidersContainer, since it can have bindings. The method from PresetManager is different, and does not contain binding information.

```python
GetBindedKeys()
```
Returns the keys in the bindings deeply dependable dictionary.

```python
GetElementsValues()
```
Returns the elements properties as a dictionary o name: value, range

```python
GetPresetManager()
```
Returns the internal preset manager for this ElementsContainer.

```python
HardSyncLFOs()
```
Restarts the phase of every running LFO signal in the container, so they move in step. Pattern signals and Timeline-synced LFOs are left as they are.

```python
ImportPresetsJSON()
```
Import stored presets to a JSON file in disk. Notice that this is a special method of ElementsContainer, since it can have bindings. The method from PresetManager is different, and does not contain bindings information.

```python
MorphAll(mode=None)
```
Morphs every element at once, each on its own timing and curve from its MorphSettings. Multi-track counterpart of MorphRandom; behaves single-track unless the engine's Multitrack gate is on.

```python
MorphGroup(tag, mode=None)
```
Morphs only the interpolatable elements whose MorphSettings.Group matches tag, each to a fresh random target on its own timing and curve. Forces the engine's per-track gate on, and elements outside the group keep their absolute start.

```python
RekeyMovedElements()
```
Re-keys this container's paths, presets and bindings onto its own elements after a rename, move or copy. Runs on load and when the container is renamed. Returns `{old path: new path}` for what moved.

```python
RenamePresetsOrder()
```
Renames the found presets in the order which they visually have. The selected preset and Blend A/B follow their presets to the new names.

```python
RepairCallbackWiring()
```
Points the embedded engine's callbacks parameter at this container's callback bridge when it is blank or wrong. Runs on load. Returns True when it rewired.

```python
ReportResult()
```
Invokes a pop-up window that displays msg and title.

```python
ResetParameters()
```
Reset all elements to the values found on drop in the ElementsContainer.

```python
RenamePresetsOrder()
```
Updates the labels in the buttons to the changed order/naming in the preset manager. Handy for when writing to the internal preset manager from outside, like with the SceneLauncher. 

```python
UpdateSize()
```
Re-scale based on elements content.

### Private

```python
constructBindingPackage(ccName, data)
```
Very low level method that assembles a dictionary compatible with TDXMorphs binding system.

```python
correctParameterType(element, uielement)
```
This function makes sure that the widget that we choose has the right parameter type. PresetManager looks at that parameter to come up with the right mapping, hence we need to parse this accordingly.

```python
createUIElement(parameter, dataSource, style=None)
```
This function allocates a particular UI from this lookup dictionary. All these are UI components that live in the UI library of TDXMorph.

```python
deleteElements()
```
Unbind and destroy all elements.

```python
getElementsOrder()
```
Find the order on which the elements are placed in the UI.

```python
getElementsPaths()
```
Gathers all e;ements in this ElementsContainer.

```python
reBind(slider, target, name, style)
```
Rebind passed slider to target, this is used when editing a bindings table.

## UI-level methods

All following methods set the elements on the UI level. These should be self explanatory if not described.

### Promoted

```python
ClearPresets()
```

```python
ExpertMode(enable=False)
```
Enables advanced features for users that are already proficient with TDMoprh. Henece "expert" mode.

```python
GetElement(elementNum)
```
Returns element in position elementNum in ElementsContainer.

```python
HardSyncLFOs()
```
Restarts every running LFO in the container at a common phase.

```python
SetPreset(int)
```
Sets the targetted preset from the UI level. A slot is a whole number from 1 (`2.0` counts as 2); anything else gives a warning.

```python
StorePreset(int)
```
Stores the targetted preset from the UI level. Same slot rule as `SetPreset`.

```python
ExportPresetsJSON()
```
Exports presets to JSON.

```python
ImportPresetsJSON()
```
Import presets to JSON.
