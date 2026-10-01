# Preset Manager

The heart of TDXMorph. This class stores and retrieves presets, and drives the morpher and
random distribution nodes.

Version 4.3.0. Capitalized methods are promoted and are the supported API. Lowercase
methods are internal and may change between versions.

For the full reference, including the preset schema, the multi-track engine and the
parameter surface, see [Documentation/PresetManager.md](../Documentation/PresetManager.md).

## Core level methods

### Properties

```python
ActivityStatus = bool (read only)
```
True while a morph is running.

```python
AutoMode = bool
```
Changes the behaviour of "randomize" and "morph" to be automatic, based on an N number of morphs.

```python
Blend = float
```
Get/set the manual blend factor between the two presets chosen in Blend A and Blend B.

```python
BlendingActive = bool
```
Get/set manual blending. Turning it on stops any running morph and seats the two chosen presets.

```python
CurrentMorphCounter = int
```
The current morphing when automatic morphing is used.

```python
CurrentPresetName = str
```
The currently selected preset. Used only as information, it does not set the preset.

```python
Lock = bool
```
Locks or opens the manager for randomization or morphing.

```python
LoopSequenceStatus = bool
```
Whether a preset sequence restarts when it reaches the end.

```python
MorphCurve = str
```
Get/set the global morph curve by name.

```python
MorphStarts = int (read only)
```
Counts every morph start this session. Read it before and after a call to tell whether a morph started, or later to tell whether another morph has started since.

```python
MorphTime = float
```
Get/set the global morph duration in seconds.

```python
MorphingType = str (read only)
```
The kind of morph currently running.

```python
NumMorphs = int
```
Number of steps used when AutoMode is on.

```python
RandomDistribution = str
```
Get/set the random distribution by name.

### Promoted

```python
ClearPresets(overwriteWarning=False)
```
Destroys all stored presets. Prompts for confirmation unless overwriteWarning is True.

```python
DeletePreset(name=None)
```
Delete the specified preset. If no name is supplied, the Target preset is deleted.

```python
ExportJSON()
```
Prompts for a file and exports all stored presets and tracked paths to JSON on disk.

```python
GetElementsPaths()
```
Returns paths to all objects associated with the manager.

```python
GetEssentialStates()
```
Go over all target operators and get their essential states for morphing. Returns None if a tracked path no longer resolves.

```python
GetEssentialStatesFrom(presetName)
```
Build essential morph states for every node stored in the given preset. A stored node that no longer exists is skipped and named in one report.

```python
GetMorphCurvesNames()
```
Returns all available morphing curve names.

```python
GetNumPresets()
```
Returns the amount of stored presets.

```python
GetPresetsKeys()
```
Returns the names of the presets.

```python
GetRandomDistributionNames()
```
Returns the possible names for random distributions.

```python
GetStates(customWarning=None)
```
Go over all target operators and get their full states, including ranges. Returns None if a tracked path no longer resolves.

```python
ImportJSON()
```
Prompts for a JSON file and loads its presets and paths. A file it cannot use is refused with the
reason and changes nothing; the import report lists any path that points at no operator.

```python
InjectPresets(presets, paths)
```
Replaces stored presets and paths with the given data, then refreshes. Accepts TouchDesigner's own stored dictionaries, so one component's presets can be handed straight to another.

```python
MigratePresets()
```
Upgrades any v1-format presets in storage to the v2 per-track schema. Idempotent. Returns the number migrated.

```python
MorphPreset(presetName=None, morphTime=None, morphCurve=None)
```
Morphs from the current state to the given preset. Without overrides each track uses the timing stored in the preset. An explicit morphTime or morphCurve applies to that call only and does not persist into the session globals.

```python
MorphRandom(mode=None)
```
Morphs parameters toward randomized target values. Respects AutoMode. mode picks the distribution
for this call only (an auto sequence keeps it for every step); to change it for good, set
Randomdistribution.

```python
OverwriteCurrentPreset()
```
Re-stores the currently targeted preset from the current parameter values.

```python
OverwritePresetsValue(item, val)
```
Overwrites one item, for example 'time' or 'curve', across all presets, all their tracks and each
preset's global block, which is what a morph reads with Multitrack off. Setting 'curve' also resets
a, b and c to that curve's defaults.

```python
OverwriteSinglePresetValue(name, item, val)
```
The same, scoped to one named preset.

```python
PlayMorphing(play=True)
```
Pauses (False) or resumes (True) the running morph where it is. A new morph or StopMorphing ends a pause; with no morph running a pause does nothing.

```python
PresetsSequence(sortKeys=False, keysSequence=None)
```
Performs the sequence of stored presets. If a keys sequence is provided it will perform the sequence in the order of that list. If sortKeys is true, it will perform the sequence in the order of the sorted keys.

```python
ReportResult(msg, title)
```
Launches a TDXMorph-formatted pop up window with the given message and title, and writes to the debug log.

```python
SetBlendingPresets(presetName, targetName)
```
Sets the blending for the specified presets. Calls the same method in the inner PresetMorpher.

```python
SetPreset(name=None)
```
Applies the specified preset immediately, with no interpolation. If no name is supplied, the Target preset is used. A morph in progress is stopped first, so the new values stick. A stored node or parameter that no longer exists is skipped and named in one report.

```python
SetRandom(mode=None)
```
Jumps all targeted parameters to a new random state, with no interpolation. Respects AutoMode. A
morph in progress is stopped first, so the new values stick. mode applies to this call only. A Menu
or StrMenu parameter gets one of its own items, drawn over the whole list.

```python
StopMorphing()
```
Stops the morph. Disarms every per-track cadence and resets the clock so the chain stops cooking. A running morph or auto sequence ends: onMorphingEnd fires with its type, and an auto sequence starts again from its first step.

```python
StorePreset(name=None, trackConfig=None)
```
Stores a preset with the given name, capturing the current state of every tracked node. If no name is supplied the one found in the custom parameters of PresetManager will be used. trackConfig optionally injects per-track timing at capture time. A path whose Filter matches no parameter is skipped and named in one report; the other paths still store.

```python
StorePresetWithData(name, data)
```
Stores a preset from an explicit data block, unlike StorePreset, which grabs the current parameter state.

```python
UpdatePaths(changedPaths)
```
Re-keys every preset's states for each {'old', 'new'} entry in the list. Keeps presets valid when a tracked node moves.

```python
updatePresetsMenu()
```
Refreshes the Target parameter menu from the stored preset names. Call after changing Presets directly.

## Private

```python
getEssentialStateFrom(targetOp, data)
```
Build essential ParamState for a target from stored param names and current values.

```python
getLockFromUI(path)
```
Check if there's an embedded UI to get locked status from, otherwise grab the status from self.

```python
getParameterSelection(target, sel, scope)
```
Return the target's parameters of the given kind whose names match the scope regex. A scope that is not a valid regex (such as `*`) is read as a wildcard pattern.

```python
getParameterValue(par)
```
Grabs the source value based on certain conditions. Some parameters, such as menus, require special treatment. This method handles that.

```python
getParams(targetOp, data)
```
Returns the filtered parameter objects for the target, per the selection.

```python
inject(preset, oldPath, newPath)
```
Re-key a preset's stored state from one path to another.

```python
normalizePathsData(paths)
```
Fill missing required per-path keys in place with the defaults used by Paths.Create.

```python
parseSelection(targetOp, data)
```
Return the selection mode: 'BUILTIN', 'CUSTOM', 'ALL' or 'NONE'.

## Removed since 3.2

`AddDataToPreset`, `RandomizeGivenParameters`, `MorphGivenParameters`, `GeneratePresetsJSON`,
`ImportPresets`, `JumpToPreset`, `GetPreset`, `GetPresetsDictionary`, `GetSavedStatesKeys`,
`GetStoredPresetsNum`, `GetEssentialStatesFromParameters`, `GetStatesFromParameters`,
`Randomize`, `RandomMorph`, `UpdatePresetsMenu` and the whole `UI*` family are gone.

Replacements: `ExportJSON` / `ImportJSON` for the JSON pair, `SetPreset` for `JumpToPreset`,
`GetPresetsKeys` / `GetNumPresets` for the key and count getters, `SetRandom` / `MorphRandom`
for the randomizers, `updatePresetsMenu` (lowercase) for the menu refresh, and
`UpdatePaths(changedPaths)` for the old single-path `UpdatePath`.
