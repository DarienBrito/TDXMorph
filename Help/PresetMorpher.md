# Preset Morpher

The morphing engine inside the PresetManager. It owns the clock, the per-track table and
the parameter writeback, and it drives all state changes of linked parameters: transitions
and randomization.

Capitalized methods are promoted and are the supported API. Lowercase
methods are internal and may change between versions.

The engine runs one chain whether the multi-track engine is on or off. With it off there is
a single `_all` track, which reproduces the 3.2 behaviour. See
[Documentation/PresetManager.md](../Documentation/PresetManager.md) for the full picture.

## Core-level methods

### Properties

```python
IsActive = bool
```
True while the morpher is active. Replaces the old ActivityStatus.

```python
MorphStarts = int (read only)
```
Counts every morph start this session. Read it before and after a call to tell whether a morph started, or later to tell whether another morph has started since.

```python
Blend = float
```
Get/Set blending factor.

```python
BlendingActive = bool
```
Get/Set blending behaviour. Turning it on stops any running morph and seats the two chosen presets into the value tables.

```python
MorphCurve = str
```
Get/set morph curve.

```python
MorphTime = float
```
Get/set morph time.

```python
NumMorphs = int
```
Amount of morphings.

```python
RandomDistribution = str
```
Get/set random distribution.

The morphing type is reported by the PresetManager's `MorphingType` property. Its values are:

  * `singleRandom`
  * `singleMorph`
  * `singlePreset`
  * `autoRandom`
  * `autoMorph`
  * `autoPresets`
  * `autoRandomWithRanges`
  * `autoMorphWithRanges`
  * `None`

### Promoted

```python
AutoRandomize(mode=None)
```
Move to N random states jumping.

```python
AutoRandomMorph(mode=None)
```
Move to N random states using morphing.

```python
ClaimGlobalShape(curve=None)
```
Record that the PresetManager's Curve a/b/n now hold the shape for `curve` (default: the current curve). Call it after writing a custom shape from a script together with a new curve.

```python
GlobalShape()
```
The global curve shape (a, b, n) that belongs to the current curve: the Curve a/b/n parameters, or the curve's defaults when the curve was changed without its shape in the same frame.

```python
GetRandomizableParameters()
GetStandardParameters()
```
The parameters the engine considers randomizable, and the ones it considers interpolatable.

```python
MorphPreset(presetName, overrideTypeFlag=False, morphTime=None, morphCurve=None)
```
Morph from the current values to the named preset. If morphTime or morphCurve is passed, the stored preset value is ignored and the passed value is used instead, for that call only. Replaces the old SetPreset on this class.

```python
MorphRandom(mode=None)
```
Morph all targeted parameters toward a new random state. `mode` picks the distribution for this call only. Replaces the old RandomMorph.

```python
PlayMorphing(play=True)
```
Pause (`False`) or resume (`True`) the running morph where it is. A pause mutes every track row and a resume un-mutes only the rows the pause muted, so a `MuteTrack` mute stays. A new morph or `StopMorphing` ends a pause; with no morph running a pause does nothing.

```python
PresetsSequence(sortKeys=False, keysSequence=None)
```
Move across all stored presets. Keys can be passed in an arbitrary order. If no keys are passed, the sequence plays in the order the items were entered.

```python
Refresh(timerActive=False)
```
Sync the value tables and idle the engine.

```python
SetBlendingPresets(presetName, targetName)
```
Allows for arbitrary blending between two presets.

```python
SetCuedSpecialValues()
```
Executes the special parameters that were saved for "end" interpolation execution, and runs any queued preset scripts when scripts are allowed.

```python
SetCurrentPresetName(name)
```
Sets current morpher preset name to the given one, here and on the PresetManager.

```python
SetRandom(mode=None)
```
Jump all targeted parameters to a new random state, with no interpolation. A morph in progress is stopped first, so the new values stick. `mode` picks the distribution for this call only. Replaces the old Randomize.

```python
SetSpecialValues()
```
Sets non-interpolatable values to the given state. Used only for preset setting, since one cannot interpolate str, ops and similar.

```python
StopMorphing(dropWriteback=False)
```
Halt the morph. Disarms every per-track cadence and resets the clock so the chain stops cooking. Pass `dropWriteback=True` when a direct write follows, so the morph's last frame cannot overwrite it. A running morph or auto sequence (also between two steps) ends here: `onMorphingEnd` fires with its type, the sequence count resets and any pause is dropped.

```python
UpdateTables()
```
Updates the target tables so values stay in sync. Typically called on completion of a timer.

### Multi-track

These exist in both modes but only do interesting work with the multi-track engine on.

```python
SetTrackConfig(config)
```
Set per-track timing and curve overrides, as `track id -> {dur, curve, a, b, c, group, endmode}`. This is a per-trigger input, not session state: the host pushes it just before each morph. MorphPreset ignores it and uses the per-track values stored in the preset, then restores the previous value when the call finishes.

```python
MorphTrack(track, mode=None)
```
Morph a SINGLE track to a fresh random target, leaving other tracks untouched.

```python
RandomizeTrack(track, mode=None)
```
Instantly randomize ONE track's parameters in place, with no morph, using its stored ranges. A Menu or StrMenu gets the item its drawn index lands on. Like `MorphTrack` and the per-track auto steps, it reads only that track, so a deleted neighbour does not stop it.

```python
MuteTrack(path, on=True)
```
Freeze or un-freeze a single track's morph progress in place. Transient: the mute is not stored in the preset.

```python
StartTrackAuto(track, mode, keys=None, randmode=None)
```
Begin a per-track auto sequence, so one track can walk its own preset or random cadence while its neighbours do something else.

```python
OnTrackComplete(track)
```
Finish one track when its morph reaches the end, leaving neighbours alone. Called by the engine.

### Callback dispatch

```python
OnMorphingStart()
OnMorphingEnd()
OnPresetCall(isMorphed)
```
Fire the corresponding user callbacks. `OnMorphingEnd` also resets the morphing type and idles the timer clock; the callback receives the type that ended.

### UI-level

```python
AutoRandomizeWithStoredRanges(mode=None)
```
Move to N random states jumping, using each element's UI-stored ranges.

```python
AutoRandomMorphWithStoredRanges(mode=None)
```
Move to N random states using morphing, with each element's UI-stored ranges.

## Private

```python
_buildTrackTable(triggered)
```
Ensure a trackTable row per active track and restart the triggered ones at the current clock value.

```python
_rebaseTrackStarts()
```
Shift every trackTable start by the clock value about to be zeroed, so a stop or an end does not leave starts in the future of a new epoch.

```python
_seatTrack(track, paramValues)
```
Rebuild the value tables preserving other tracks, seating the given one fresh.

```python
getParameterValue(data)
```
Return a random value for the parameter described by data.

```python
getRandomValue(minMax, paramType, decimalPoints=8)
```
Gets random values with different distributions from the RandomGenerator module. Certain parameter types need only certain values, such as two integer states.

```python
keepCount(funcOn=None, funcOff=None)
```
Run funcOn each step until NumMorphs is reached, then funcOff, then reset.

```python
setAttributeParameterProperties(states)
```
Low level method to write parameters directly.

```python
setComputedParameterProperties(states, target)
```
Low level method to write computed parameters.

```python
setOriginalParameterProperties(states, target)
```
Low level method to write original parameters.

```python
setRandomVals(states=None)
```
Randomizes values using the defined random distribution, writing directly to the parameters. Used by SetRandom.

```python
setTimer(active=True, triggeredTracks=None)
```
Rebuild the per-track progress table and set the active flag.

```python
writeEssentialValues(presetName)
```
Fill currentValues with the start state. Returns whether any state existed.

```python
writePresetToChannels(target, preset)
```
Write a preset's interpolatable values to the target and stack the rest as specials. Non-interpolatable parameters go to the Special table.

```python
writePresetValues(target, presetName, withSpecial=True)
```
Load a preset into the target. Returns False if the preset does not exist.

```python
writeRandomVals(states=None)
```
Fill newValues with random morph targets, according to the selected random distribution.

## Removed since 3.2

`ActivityStatus` is now `IsActive`. `Randomize` is now `SetRandom`, `RandomMorph` is now
`MorphRandom`, and `SetPreset` on this class is now `MorphPreset`.
`MorphGivenParameters`, `RandomizeGivenParameters`, `RandomizeWithStoredRanges`,
`RandomMorphWithStoredRanges`, `getStatesFromGivenParameters`, `inferRandMode`,
`signalCompletion` and the inert `enableInterpolation` are gone. Completion is now signalled through the trackDone chain rather
than by flipping a constant.
