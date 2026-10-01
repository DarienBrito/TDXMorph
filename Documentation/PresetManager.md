# Preset Manager

**Part of the TDXMorph Toolkit**  
Copyright © 2020–2026  
**Author:** [Darien Brito](https://www.darienbrito.com)  
**License:** [MIT License](https://opensource.org/license/mit)  
**Version:** 4.3.0

---

## Overview

`extPresetManager` is the **core engine** of the TDXMorph system for **TouchDesigner**.  
It manages the storage, retrieval, morphing and randomization of parameter presets across multiple operators.

Unlike the UI components (`extElementsContainer`, `extSceneLauncher`), this class operates entirely at the **data and logic level**, following a **Model-View-Controller (MVC)** structure. It acts as the **Model**, holding the preset data and driving the `PresetMorpher` and `RandomGenerator` nodes.

The component has no UI of its own. You drive it from its custom parameters, from Python, or by attaching one of the front ends (ParameterMorpher, SceneLauncher, PresetInspector).

---

## What changed in 4.x

Version 4 is a substantial rewrite of the morphing engine. If you are coming from 3.2, read this section first.

- **Multi-track engine.** Every tracked node can morph on its own clock, with its own duration, curve, end mode and group. Enable it with the **Multi-Track Engine** toggle on the Options page. It is **off by default**, which reproduces the 3.2 single-track behaviour, so existing projects keep working unchanged.
- **Preset schema v2.** Timing fields moved from the top of a preset down into each tracked path, which is what makes per-track timing possible. Old presets are migrated automatically (see [Preset format](#preset-format)).
- **Per-track auto sequences.** `StartTrackAuto` lets one node walk its own preset or random cadence while its neighbours do something else entirely.
- **End modes.** Each track can hold, loop, ping-pong or advance when it reaches the end of its morph.
- **Curve shapes.** Curves take three shape coefficients (`a`, `b`, `n`), stored per track and seeded from each curve's own defaults.
- **Blending.** Two presets can be loaded into the engine and crossfaded manually with a single factor, independently of the timed morph.
- **Owned Paths editor.** The paths list is now the MIT `ListView` widget rather than a palette Lister, so the component carries no third-party content.

### Removed since 3.2

`RandomizeGivenParameters`, `MorphGivenParameters` and `AddDataToPreset` no longer exist. `UpdatePath(oldPath, newPath)` has been replaced by `UpdatePaths(changedPaths)`, which takes a list and re-keys every preset in one pass.

---

## Table of Contents

- [Architecture](#architecture)
- [Initialization](#initialization)
- [Custom parameters](#custom-parameters)
- [Stored properties](#stored-properties)
- [Preset format](#preset-format)
- [Properties](#properties)
- [Public methods](#public-methods)
  - [Preset management](#preset-management)
  - [Import and export](#import-and-export)
  - [Morphing and randomization](#morphing-and-randomization)
  - [State retrieval](#state-retrieval)
  - [Path handling](#path-handling)
  - [Utility](#utility)
- [The multi-track engine](#the-multi-track-engine)
- [Callbacks](#callbacks)
- [Curves and distributions](#curves-and-distributions)
- [Tracked paths](#tracked-paths)
- [Design philosophy](#design-philosophy)
- [Dependencies](#dependencies)

---

## Architecture

| **Attribute** | **Description** |
|----------------|-----------------|
| **Class** | `extPresetManager` |
| **Role** | Core preset logic for saving, recalling, morphing and randomizing TouchDesigner parameters. |
| **Type** | Data Model |
| **Design Pattern** | Part of the TDXMorph MVC system (acts as the Model). |
| **Usage Context** | Parameter management in presets, morphing systems and automated transitions. |
| **License** | MIT |

Internal nodes:

| **Node** | **Role** |
|---|---|
| `PresetMorpher` | The morphing engine. Owns the clock, the per-track table and the parameter writeback. |
| `RandomGenerator` | Random value generation across selectable distributions. |
| `Paths` | The database of tracked operator paths and their per-path settings. |

---

## Initialization

```python
manager = op('PresetManager')
```

All public methods are promoted, so they are reachable directly on the component:

```python
op('PresetManager').StorePreset('intro')
op('PresetManager').MorphPreset('intro')
```

---

## Custom parameters

### Storing

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Presetname` | Str | `''` | Name used by Store, Delete and Overwrite when no name is passed. |
| `Storepreset` | Pulse | | Capture the current state of every tracked node into a preset. |
| `Clearpresets` | Pulse | | Delete all presets (asks for confirmation). |
| `Searchin` | Menu | `Wholenetwork` | Scope for the tracked-node scan: whole network, or one operator. |
| `Searchoperator` | OP | `''` | The operator to search inside when `Searchin` is `Specificoperator`. |
| `Pathseditor` | Pulse | | Open the paths editor. |
| `Pathsupdate` | Pulse | | Re-scan for tracked nodes that have moved and re-key their presets. Copies of a node still in place are ignored; a path several nodes claim is left alone and they are named. |
| `Pathsclear` | Pulse | | Remove every tracked path and strip its TDXMorph tag. |

### Triggering

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Target` | Menu | | The preset acted on by Set, Morph, Delete and Overwrite. Rebuilt from stored preset names. |
| `Set` | Pulse | | Apply the target preset instantly, with no interpolation. |
| `Morph` | Pulse | | Morph from the current state to the target preset. |
| `Stop` | Pulse | | Stop the active morph. |
| `Deletepreset` | Pulse | | Delete the target preset. |
| `Overwritepreset` | Pulse | | Re-store the target preset from the current parameter values. |
| `Morphtime` | Float | `2.0` | Global morph duration in seconds. |
| `Morphcurve` | Menu | `Linear` | Global interpolation curve. |
| `Curvea` / `Curveb` / `Curven` | Float | `0.0` / `1.0` / `1.0` | Shape coefficients for the selected curve. Picking a new curve resets them to that curve's defaults; a curve always morphs and stores with its own shape, even when a script changes `Morphcurve` and morphs in the same frame. |
| `Randomdistribution` | Menu | `Uniform` | Distribution used for random values. |
| `Setrandom` | Pulse | | Jump every targeted parameter to a new random state. |
| `Morphrandom` | Pulse | | Morph every targeted parameter toward a new random state. |
| `Automode` | Toggle | `Off` | Make Morph and Random run a sequence of `Morphings` steps instead of one. |
| `Morphings` | Int | `2` | Number of steps when `Automode` is on. |
| `Presetssequence` | Pulse | | Step through the stored presets in order. |
| `Loopsequence` | Toggle | `Off` | Restart the sequence when it reaches the end. |

### Options

| **Parameter** | **Type** | **Default** | **Description** |
|---|---|---|---|
| `Multitrack` | Toggle | `Off` | Enable the multi-track engine. Off reproduces 3.2 single-track behaviour. |
| `Lock` | Toggle | `Off` | Lock the manager against randomization and morphing. |
| `Allowscripts` | Toggle | `On` | Allow embedded per-preset scripts to run. |
| `Blendactive` | Toggle | `Off` | Enable manual blending between two presets. |
| `Blenda` / `Blendb` | Menu | | The two presets to blend. |
| `Blendfactor` | Float | `0.0` | Crossfade position between A and B. |
| `Trackingtag` | Str | `TDXMorphPath` | The tag written onto tracked nodes so they can be found after a move. |
| `Importjson` / `Exportjson` | Pulse | | Load or save presets and paths as JSON. |

### Info (read only)

`Currentlystored`, `Currentmorphcount`, `Currentpreset` and `Morphingtype` report engine state for display and for binding.

---

## Stored properties

| **Property** | **Type** | **Description** |
|---------------|----------|-----------------|
| `Presets` | `dict` | Every stored preset, keyed by name. See [Preset format](#preset-format). |

Tracked paths live in their own storage on the `Paths` node, not inside `Presets`.

---

## Preset format

`PRESET_FORMAT_VERSION = 3`. The version is written into exported JSON so future loaders can detect older files.

A preset is a dict with two keys. `global` holds the settings that were live on the component when the preset was stored, and `states` maps each tracked operator path to its captured parameters plus that track's own morph settings:

```python
{
  'global': {                     # captured from the live pars at store time
    'time': 2.0,                  # morph duration used when Multitrack is OFF
    'curve': 'Linear',
    'a': 0.0, 'b': 1.0, 'c': 1.0,
    'distr': 'Uniform',
  },
  'states': {
    '/project1/noise1': {
      'params': [ {...}, {...} ],   # one entry per captured parameter
      'time': 2.0,                  # this track's morph duration, in seconds
      'curve': 'Linear',            # this track's interpolation curve
      'a': 0.0, 'b': 1.0, 'c': 1.0, # curve shape coefficients
      'distr': 'Uniform',           # this track's random distribution
      'group': '',                  # optional group label
      'endmode': 'hold',            # hold | loop | pingpong | advance
      'quantize': 0,                # optional quantization
    }
  }
}
```

Each entry in `params` is a serialized `ParamState`:

```python
{'paramName': 'period', 'value': 1.0, 'type': 'Float', 'isOP': False, 'locked': False,
 'normMin': 0.0, 'normMax': 1.0, 'min': 0.0, 'max': 10.0}
```

`locked` is for display only. It records a ParameterMorpher element's own Lock at store time and is never read on recall: the live Lock decides what a preset may change.

The four range keys are present only for a **full** capture (`StorePreset`). Transient *essential* captures, used for the start state of a morph, leave them out.

### Which timing a recall uses

Both sets are always stored. Which one drives a morph depends on the `Multitrack` gate:

| `Multitrack` | Morph runs on | Ignores |
|---|---|---|
| OFF (global) | the preset's `global` block | the per-track values |
| ON (per track) | each track's own `time`, `curve`, `distr` | the `global` block |

Because the global values are stored with the preset, a global-mode recall runs on the timing that was live **when you stored it**, not the timing that happens to be set now, and two presets can carry different global timings. Nothing reads track order.

### Reserved key

Do not add a top-level `time` key to a preset. It is how a v1 preset is detected, and the animation builder gives a top-level `time` precedence over the per-state value. The preset-level settings are nested under `global` for exactly that reason.

### Migrating from older versions

Version 1 kept the timing at the top of the preset rather than per track:

```python
{'states': {path: [params]}, 'time': 2.0, 'curve': 'Linear', 'distr': 'Uniform'}
```

Version 2 moved it per track but had no preset-level block at all, so a global-mode recall fell back to whichever track came first.

`MigratePresets()` upgrades both in place and is **idempotent**, so calling it twice is harmless. v1 is detected by a top-level `time` key: each track inherits the preset's old global time, curve and distribution, `a`/`b`/`c` are seeded from that curve's defaults, and the new fields take their defaults (`group` empty, `endmode` `hold`, `quantize` 0). A v2 preset gains a `global` block seeded from its first track, which is exactly what a v2 build morphed on, so **a migrated preset behaves identically** until you re-store it.

Compatibility runs both ways. A v3 preset loaded by a v2 build has no top-level `time`, so that build skips migration, reads `states` as before and ignores the block it does not recognise.

```python
n = op('PresetManager').MigratePresets()
print(f'{n} presets migrated')
```

---

## Properties

```python
AutoMode = bool
```
Makes randomize and morph run automatically for `NumMorphs` steps.

```python
ActivityStatus = bool (read only)
```
True while a morph is running.

```python
Blend = float
```
Get/set the manual blend factor between the two chosen presets.

```python
BlendingActive = bool
```
Get/set manual blending. Turning it on stops any running morph and seats the two chosen presets into the engine tables.

```python
CurrentMorphCounter = int
```
The index of the current morph when automatic morphing is running.

```python
CurrentPresetName = str
```
The currently selected preset. Informational only: setting it does not apply the preset.

```python
Lock = bool
```
Locks or opens the manager for randomization and morphing.

```python
LoopSequenceStatus = bool
```
Whether a preset sequence restarts when it reaches the end.

```python
MorphCurve = str
```
Get/set the global morph curve by name.

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
Number of steps used when `AutoMode` is on.

```python
RandomDistribution = str
```
Get/set the random distribution by name.

---

## Public methods

### Preset management

```python
StorePreset(name=None, trackConfig=None)
```
Capture the current parameter state of every tracked node into a preset. Falls back to the `Presetname` parameter when `name` is omitted. `trackConfig` is an optional `path -> {time, curve, distr, ...}` mapping that lets a host inject per-track fields at capture time; any key you leave out falls back to the global value. When `trackConfig` is `None`, time, curve and shape are captured from the paths editor, and an existing preset keeps its other per-track fields (distribution, group, end mode), so re-storing does not flatten them. A path whose Filter matches no parameter is skipped and named in one report; the other paths still store.

```python
StorePresetWithData(name, data)
```
Store a preset from an explicit data block instead of from the current state. `data` must match the [preset format](#preset-format).

```python
SetPreset(name=None)
```
Apply a stored preset's values immediately, with no interpolation. A morph in progress is stopped first, so the new values stick.

```python
DeletePreset(name=None)
```
Delete a preset by name. Falls back to the `Presetname` parameter.

```python
ClearPresets(overwriteWarning=False)
```
Delete every preset. Prompts for confirmation unless `overwriteWarning` is True.

```python
OverwriteCurrentPreset()
```
Re-store the currently targeted preset from the current parameter values.

```python
OverwritePresetsValue(item, val)
```
Overwrite one field across **all** presets, all their tracks and each preset's global block, which is what a morph reads with Multitrack off. Useful for retiming a whole set at once, for example `OverwritePresetsValue('time', 4.0)`. Setting `curve` also resets `a`, `b` and `c` to that curve's defaults.

```python
OverwriteSinglePresetValue(name, item, val)
```
The same, scoped to one named preset.

```python
GetPresetsKeys()
GetNumPresets()
```
The stored preset names, and how many there are.

```python
MigratePresets()
```
Upgrade any v1 presets in storage to the v2 per-track schema. Idempotent. Returns the number migrated.

```python
updatePresetsMenu()
```
Refresh the `Target` parameter menu from the stored preset names. Call this after changing `Presets` directly.

### Import and export

```python
ExportJSON()
ImportJSON()
```
Prompt for a file and export or load presets plus tracked paths as JSON. Import refuses a file it cannot use (unreadable, a newer format, another tool's file such as a PresetSnap export, or presets of the wrong shape), says why, and changes nothing. After an import, the report lists any path that points at no operator in this project; edit or remove those in the paths editor.

```python
InjectPresets(presets, paths)
```
Replace stored presets and paths with the given data, then refresh. This is the programmatic equivalent of an import, and it accepts TouchDesigner's own stored dictionaries, so you can hand one component's presets straight to another.

### Morphing and randomization

```python
MorphPreset(presetName=None, morphTime=None, morphCurve=None)
```
Morph from the current state to `presetName`. Without overrides, each track uses the timing stored in the preset. An explicit `morphTime` or `morphCurve` wins over the stored values for that call only, and does not persist into the session globals.

```python
SetRandom(mode=None)
MorphRandom(mode=None)
```
Randomize the targeted parameters, either instantly or through a morph. Both respect `AutoMode`, running a single step or a sequence of `NumMorphs`. A morph in progress is stopped first. `mode` picks the distribution for this call only, and an auto sequence keeps it for every step; to change it for good, set `Randomdistribution`.

```python
PresetsSequence(sortKeys=False, keysSequence=None)
```
Step through the stored presets in order. Pass `sortKeys` to sort by name, or `keysSequence` for an explicit order.

```python
SetBlendingPresets(presetName, targetName)
```
Load two presets into the engine tables for manual blending with `Blend`.

```python
StopMorphing()
```
Stop the active morph. Disarms every per-track cadence and resets the clock so the chain stops cooking.

```python
PlayMorphing(play=True)
```
Play or pause the morph clock.

### State retrieval

```python
GetStates(customWarning=None)
```
Full parameter states, including ranges, for every tracked node. Returns `None` if a tracked path no longer resolves.

```python
GetEssentialStates()
GetEssentialStatesFrom(presetName)
```
Lightweight states without ranges, for all tracked nodes or for the nodes stored in one preset.

```python
GetElementsPaths()
GetMorphCurvesNames()
GetRandomDistributionNames()
```
The tracked paths, the available curve names and the available distribution names.

### Path handling

```python
UpdatePaths(changedPaths)
```
Re-key every preset's states for each `{'old': ..., 'new': ...}` entry in the list. This is what keeps presets valid when you move a tracked node.

### Utility

```python
ReportResult(msg, title)
```
The canonical reporter: prints, writes to the debug log and shows a dialog.

---

## The multi-track engine

With **Multi-Track Engine** off, all tracked nodes share one clock and one duration. This is the 3.2 behaviour and remains the default.

With it on, each tracked path becomes a **track** with its own row in the engine's track table: its own start time, duration, curve, shape coefficients, group and end mode. Tracks start, run and complete independently.

Per-track fields are captured into the preset by `StorePreset`, and can be edited per path in the paths editor once the toggle is on.

```python
pm = op('PresetManager')
morpher = pm.op('PresetMorpher')

# per-track overrides for the next random morph
morpher.SetTrackConfig({
    '/project1/noise1': {'dur': 4.0, 'curve': 'Scurve'},
    '/project1/noise2': {'dur': 1.0, 'curve': 'Easeout', 'endmode': 'loop'},
})
pm.MorphRandom()
```

`SetTrackConfig` is a **per-trigger input, not session state**. The host pushes it just before each morph. A preset morph does not use it: `MorphPreset` morphs each track with the values stored in the preset (time, curve, shape, group and end mode), then restores the previous config when the call finishes. To give a preset its own per-track values, store it with them. A plain Store takes time, curve and shape from the paths editor. A script can set every per-track field, end mode and group included, with `StorePreset(name, trackConfig={path: {'time': 2.0, 'curve': 'Linear', 'endmode': 'loop'}})`; any field it leaves out falls back to the global value. To inspect what a track is actually morphing with, read its row in the engine's `trackTable` rather than reading the config back.

### End modes

| **Mode** | **Behaviour at the end of a track's morph** |
|---|---|
| `hold` | Stay at the target values. The default. |
| `loop` | Restart the track from its start values. |
| `pingpong` | Swap start and target, then run again. |
| `advance` | Move on to the next step of the track's auto sequence. |

### Per-track methods

These live on the `PresetMorpher` node:

```python
MorphTrack(track, mode=None)        # morph one track to a fresh random target
RandomizeTrack(track, mode=None)    # randomize one track in place, no morph
MuteTrack(path, on=True)            # freeze one track's progress where it stands
StartTrackAuto(track, mode, keys=None, randmode=None)  # begin a per-track auto sequence
SetTrackConfig(config)              # per-trigger timing and curve overrides
```

`MuteTrack` is transient: the mute is not stored in the preset.

---

## Callbacks

The **Callbacks** page points at a DAT, by default `./presetMorpher_callbacks`. Three hooks are dispatched:

```python
def onMorphingStart(presetManager, morphingType, presetName):
	"""Called when any morph starts."""
	return

def onMorphingEnd(presetManager, morphingType, presetName):
	"""Called when any morph ends."""
	return

def onPresetCall(presetManager, morphingType, presetName):
	"""Called only when a preset is explicitly invoked."""
	return
```

`onMorphingStart` and `onMorphingEnd` fire for every morph, including random ones. With Multi-Track Engine on, a track in `loop` or `pingpong` never finishes, so `onMorphingEnd` (and anything chained to it, such as SceneLauncher follow actions) waits until the morph is stopped. `onPresetCall` fires only when a named preset is invoked, which is the hook you want for cueing and follow actions.

---

## Curves and distributions

Curves are registered in `PresetMorpher/curveLib`, which also holds each curve's default shape coefficients. The `Morphcurve` menu exposes:

`Linear`, `Scurve`, `Doubleoddpolynomial`, `Quadraticviapoint`, `Exponentialeasing`, `Easein`, `Easeout`,
`Sigmoid`, `Circulareasein`, `Circulareaseout`, `Sine`, `Rect`, `Tri`, `Random`, `SnapIn` and `SnapOut`.

That is all **16** shapes the registry holds.

Distributions come from `RandomGenerator`: `Uniform`, `Normal`, `Beta`.

```python
lib = op('PresetManager/PresetMorpher/curveLib').module
lib.shape('Scurve', 0.5, 0.0, 1.0, 1.0)   # shape a 0..1 progress value
lib.defaults('Scurve')                     # that curve's (a, b, n) defaults
```

---

## Tracked paths

The `Paths` node holds the database of operators the manager watches, along with each one's per-path settings. Nodes are marked with the `Trackingtag` value (`TDXMorphPath` by default), which is how a moved node is found again.

```python
paths = op('PresetManager/Paths')
paths.Create('/project1/noise1')     # register a node
paths.Delete('/project1/noise1')     # unregister it and strip its tag
paths.AutoUpdatePaths()              # reconcile moved nodes, returns the changes applied
paths.GetPathsKeys()                 # every tracked path
paths.OpenUI()                       # open the paths editor
```

Drag an operator onto the paths editor to register it. Double-click a row to open that node in the View pane. Right-click a column header to rename or realign it, and right-click empty space for the general Update and Clear menu.

The editor is a `ListView` instance (the same MIT widget shipped standalone in this repository), so the per-path Time, Curve, a, b and n columns appear only when the multi-track engine is on.

## Design Philosophy

- **The engine idles clean.** The clock is a Timer CHOP that runs only while morphing, so nothing in the chain cooks at rest.
- **UI independence.** Core logic is fully detached from any user interface.
- **Data transparency.** Presets are stored as clean, JSON-serializable dictionaries, versioned and migrated forward automatically.
- **The Model owns the data.** UI components read and drive the manager; they never hold preset state of their own.
- **Everything public is promoted.** Capitalized methods are the supported API. Lowercase methods are internal and may change between versions.

---

## Dependencies

### TouchDesigner components
- `PresetMorpher`
- `Paths`
- `RandomGenerator`
- `op.TDResources` (TouchDesigner's own pop-up menu and dialog host, present on every install; the component degrades gracefully when it is absent)

The `.tox` is otherwise self-contained: no palette components, no external files and no third-party content.

### Python libraries
- `TDStoreTools.StorageManager`
- `re`
- `json`
- `itertools`
- `sys`
