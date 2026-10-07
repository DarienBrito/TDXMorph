# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Since version 4, the free components carry their own version numbers rather than a single
toolkit version, because they now ship and update independently.

## [Open Toolkit 4.3.6] (4th quarter of 2026)

Current component versions on this line: PresetManager **4.4.4**, PresetSnap **1.3.0**,
ControlMapper **1.1.0**, PresetInspector **1.5.8**, JSONTree **1.5.7**, ListView **1.0.9**.
This list tracks what the line ships today, so it moves with every component release
rather than recording what any one toolkit release contained.

### PresetManager 4.4.4

- **Stop freezes a morph where it is.** Stopped past the halfway point, a morph snapped to its target, and
  stopped after a Pause it jumped back to its start. Stop now leaves every value where it was.
- **A track set to `loop` or `pingpong` keeps looping** when another track finishes in the same frame. With
  equal durations the whole morph used to end. **Behaviour change.**
- **Manual Blend moves Int, Toggle and Menu values too:** preset A's below a factor of 0.5, preset B's from
  0.5 up. They never moved before. **Behaviour change.**
- **A track stored on its own curve keeps that curve's shape** instead of taking the global curve's.
- **Export JSON keeps your preset order.** It sorted the names, so an export and import reordered Presets
  Sequence.

## [Open Toolkit 4.3.5] (4th quarter of 2026)

PresetManager **4.4.3**, PresetSnap **1.3.0**, ControlMapper **1.1.0**, PresetInspector **1.5.8**,
JSONTree **1.5.7**, ListView **1.0.9**.

### JSONTree 1.5.7

- Leftover expression text from an old file layout is gone from the toolbar. Nothing changes in use.

### PresetInspector 1.5.8

- Carries JSONTree 1.5.7's cleanup, and its embedded tree no longer holds leftover expression text naming a
  development folder. Nothing changes in use.

## [Open Toolkit 4.3.4] (4th quarter of 2026)

PresetManager **4.4.3**, PresetSnap **1.3.0**, ControlMapper **1.1.0**, PresetInspector **1.5.7**,
JSONTree **1.5.6**, ListView **1.0.9**.

### PresetManager 4.4.3

- The morph engine's value stores ship empty instead of holding the channels of its last run.
  Nothing changes in use.

## [Open Toolkit 4.3.3] (4th quarter of 2026)

PresetManager **4.4.2**, PresetSnap **1.3.0**, ControlMapper **1.1.0**, PresetInspector **1.5.7**,
JSONTree **1.5.6**, ListView **1.0.9**.

### PresetManager 4.4.2

- The morph engine ships with no state left from its last run.
- The Paths editor no longer carries two unused parameter pages left from the Lister it replaced.
- Nothing changes in use.

## [Open Toolkit 4.3.2] (4th quarter of 2026)

PresetManager **4.4.1**, PresetSnap **1.3.0**, ControlMapper **1.1.0**, PresetInspector **1.5.7**,
JSONTree **1.5.6**, ListView **1.0.9**.

### PresetManager 4.4.1

- The Paths editor no longer carries an unused table that held a path from the development project.
  Nothing read it, so nothing changes in use.

## [Open Toolkit 4.3.1] (4th quarter of 2026)

PresetManager **4.4.0**, PresetSnap **1.3.0**, ControlMapper **1.1.0**, PresetInspector **1.5.7**,
JSONTree **1.5.6**, ListView **1.0.9**.

### PresetInspector 1.5.7

- **Behaviour change: `Presetmanager` starts empty.** It defaulted to the name `PresetManager`, so a
  fresh PresetInspector attached itself to any sibling called that. It now starts unattached, like every
  other TDXMorph component; point it at a PresetManager yourself.

## [Open Toolkit 4.3.0] (4th quarter of 2026)

PresetManager **4.4.0**, PresetSnap **1.3.0**, ControlMapper **1.1.0**, PresetInspector **1.5.6**,
JSONTree **1.5.6**, ListView **1.0.9**.

The release audit: every component was reviewed ahead of the TDXMorph 4 release. Items marked
**Behaviour change** alter what an existing setup does, not only what a broken one does. PresetManager, JSONTree and
PresetInspector now read and write files as UTF-8, so non-ASCII names survive and a file saved with
a BOM (Notepad) loads.

### PresetManager 4.4.0

Morphing:

- **A morph with nothing to interpolate ends.** With only Int, Toggle or Menu parameters tracked,
  with Lock on, or on a fresh engine in Automode, the morph never completed and the clock ran forever.
- **Pause works.** `PlayMorphing(False)` freezes the running morph where it is and
  `PlayMorphing(True)` continues from there. A new morph or Stop ends a pause. **Behaviour change:**
  `PlayMorphing` no longer touches the clock's `play` flag.
- **Stop ends the morph properly.** It fires `onMorphingEnd` for a running morph or auto sequence
  (also between two steps), and an auto sequence starts again from its first step. `onMorphingEnd`
  reports the type that ended instead of `None`, and a random morph reports `None` as its preset name.
  **Behaviour change:** callbacks see `onMorphingEnd` on Stop, on Set and Set Random during a morph,
  and on turning Blend Active on during one.
- **Set and Set Random during a running morph stick.** The morph's next frame used to overwrite them;
  both now stop the morph first.
- **Automode and Presets Sequence keep going with Multitrack on.** They stopped after one step.
- **The live Lock decides a preset morph.** A preset stored with Lock on, recalled with Lock off,
  jumped to the target and slid to 0.0. The stored lock is no longer read, and Store no longer saves
  it. **Behaviour change:** with Lock on, a preset morph also leaves Menu, Toggle and Int values alone.
- **A preset's stored end mode applies on recall.** **Behaviour change:** with Multitrack on, a preset
  whose tracks were stored as `loop` or `pingpong` keeps looping after recall, so it no longer fires
  its morph end by itself. Presets with no stored end mode still hold.
- **A curve always morphs with its own shape.** A curve passed to `MorphPreset` with Multitrack off
  used the preset's stored shape, so Easein, Easeout and Exponentialeasing finished within a few
  frames (every SceneLauncher launch passes one). A script that set `Morphcurve` and then stored or
  morphed in the same frame used the previous curve's shape. Both now use the new curve's defaults,
  and a shape you tune afterwards is kept. New: `PresetMorpher.GlobalShape()` and
  `ClaimGlobalShape(curve)`.
- **What you pass to one call stays with that call.** A `morphTime` or `morphCurve` passed to
  `MorphPreset`, and a `mode` passed to `SetRandom`, `MorphRandom`, `MorphTrack` and the other random
  calls, used to overwrite Morph Time, Morph Curve or Random Distribution for good. An auto sequence
  started with a `mode` keeps it for all its steps. **Behaviour change:** set `Morphtime`,
  `Morphcurve` or `Randomdistribution` to change the settings themselves.
- **Manual blending no longer changes your morph settings.** Turning Blend Active on left the second
  preset's time, curve, distribution and per-track timing on the session.
- **Random values stay valid.** `Normal` could land outside the range (about 1 draw in 22). A StrMenu
  got a number instead of one of its items, and a Menu only reached its first two items.
- **A deleted node no longer opens a blocking dialog.** Morph and Set skip a node or parameter you
  deleted, name it in one report and still morph or set the rest. Automode Morph Random reports it and
  starts nothing, and one deleted node no longer stops every per-track cadence. **Behaviour change:**
  the dialog no longer offers to remove the path; Delete or Clear in the Paths editor does that.
- **Set Random after a stopped morph keeps its values** with Automode on.
- **Overwriting a preset value reaches the morph.** `OverwriteSinglePresetValue` and
  `OverwritePresetsValue` never wrote the global block a morph reads with Multitrack off, so a
  SceneLauncher Length or Curve edit changed nothing. A new curve also re-seeds its shape.
- **`SetPreset()` and `DeletePreset()` with no name act on Target**, as the Set and Delete pulses do.
- **New: `MorphStarts`**, a read-only count of morph starts, so a caller can tell whether its own
  morph started and whether another has started since.

Paths:

- **Deleting or clearing paths asks what to do with the presets that store them:** Keep (the old
  behaviour, on Enter), Remove (drop their values from every preset) or Cancel (on Esc).
  **Behaviour change:** Clear's confirm is now non-blocking like Delete's, so `Paths.Clear()` without
  `overwriteWarning=True` returns before the paths are cleared. New: `Delete(..., prune=False)`,
  `Clear(..., prune=False)`, `PresetsUsing(paths)`, `PruneFromPresets(paths)`, `ConfirmRemove(...)`
  and an `escButton` argument on `StandardPopDialog`.
- **Editing a path can no longer lose data.** An edit onto a path with no operator, or one already
  tracked, is refused and reported. A valid edit moves the tracking to the new node.
- **A Filter that matches nothing skips only its own path** and names it, instead of making every
  Store a silent no-op and every morph abort. A Filter that is not a valid regex (such as `*`) reads
  as a wildcard. **Behaviour change:** Store is partial on such a path, not all-or-nothing.
- **Copying a tracked node no longer steals its tracking.** Copies are ignored; a plain move or
  rename is still followed, and Update Paths keeps a moved path's own time, curve and shape.
- **Picking a curve for a path resets its a/b/c** to that curve's defaults. **Behaviour change:**
  `Paths.OverwriteItem(path, 'curve', ...)` re-seeds a/b/c when the curve changes; setting the same
  curve keeps a tuned shape.
- **The Paths editor follows Import JSON and Update Paths**, and a drag in an out-of-date editor no
  longer deletes the paths it was not showing.
- **Changing Tracking Tag keeps the moves tracked.** New: `Paths.Retag(oldTag, newTag)`.

Files:

- **Importing a PresetSnap export can no longer wipe everything.** A file stamped by another tool is
  refused, and an unstamped file of the wrong shape is refused before anything is cleared. New:
  `importFile(fileName)`; `ImportJSON` keeps the file dialog.
- **Preset files from builds before 3.2 morph and set after import.**
- **Import names the paths that point at nothing** in this project.

### PresetSnap 1.3.0

- **A preset named `param`, `include` or `mode` no longer overwrites the table.** Those are the
  table's own column names; such a name now becomes `param 2` and so on.
- **A host page you already called `Presets` keeps your parameters.** They are captured like any
  other, and Remove Page takes off only PresetSnap's own controls.
- **Behaviour change: a parameter missing at a rescan stays included.** It is marked `gone` and kept
  on, and the next rescan after it returns picks it up. It used to be switched off for good.
- **A StrMenu parameter recalls a typed value** that is not one of its menu items.
- **The status line says more.** Refusals (a clone host, no host, a refused import) show there, and a
  recall counts what failed and what is gone, for example `A: set 5, 1 failed, 2 gone`.
- **A host cloned by an expression is refused** like any clone, so clone-sync cannot wipe the page.
- **Importing another TDXMorph tool's file is refused**, naming the tool that wrote it. PresetSnap
  files from before 1.2.0 still load.
- **Rescan is faster on large hosts.**
- **Docs corrected:** values inside a sequence are captured (each block's values, such as
  `const0value`); only the block count is not.
- Its parameter editor carries the ListView 1.0.9 fixes.

### ControlMapper 1.1.0

ParameterMorpher and SceneLauncher carry these through their embedded copy.

- **Pickup works on a parameter outside the mapping's range.** It never engaged; it now engages when
  the control reaches that end of its travel.
- **Behaviour change: pickup re-arms when something else moves the parameter.** After a preset
  recall, a morph or a hand edit, the control waits until it reaches the new value instead of
  jumping the parameter back.
- **Pickup works on a menu**, waiting for the control to reach the selected item.
- **Re-learning a mapping keeps its range, takeover and place in the list**, and only takes the new
  channel.
- Its mapping editor carries the ListView 1.0.9 fixes.

### ListView 1.0.9

Every component that embeds a ListView carries these.

- **The selection stays on its row when the rows change.** Rows with a `_key` keep their selection by
  key through `Refresh` and `SetRows`. Rows without a `_key` behave as before.
- **A column without a key keeps its own width, alignment and label** until the next rebuild, instead
  of passing them to every other keyless column. Give the column a `key` to keep a change.
- **Two rows with the same name are no longer one row for a double-click.**
- **A whole-number value cell accepts a fraction**, and the value becomes a decimal.

### JSONTree 1.5.6

- **Editing a file keeps its non-ASCII text.** A name like `café ✓` came back as `cafÃ© âœ“`.
- **A chosen skin survives reopening the project.** Hand-edited Look colours are not touched.
- **A saved search filters the tree on reopen.**
- **A whole-number value accepts a fraction**; a whole number typed over an integer stays an integer.
- **One edit parses the tree once** instead of up to three times.

### PresetInspector 1.5.6

Carries the JSONTree 1.5.6 fixes, and **the inspector's Search shows in the tree's search field**.

## [Open Toolkit 4.2.1] (3rd quarter of 2026)

PresetManager **4.3.0**, PresetSnap **1.2.1**, ControlMapper **1.0.5**, PresetInspector **1.5.5**,
JSONTree **1.5.5**, ListView **1.0.8**.

A new free component, per-preset timing in PresetManager, and a text rendering fix in ListView
that reaches every component embedding it.

### PresetSnap 1.2.1

It ships with an empty parameter list again.

- **The 1.2.0 download carried seven leftover rows**, one per parameter of the project it was
  exported from, each marked `missing` with `include` off. A fresh drop-in opened Edit Params on
  dead rows that `Rescan` cannot clear, since a vanished parameter is marked rather than deleted
  by design. No values were exposed. The export now clears the table to its header.

### ControlMapper 1.0.5

A mapping never overwrites a parameter that something else is driving.

- **A parameter driven by an expression or an export is refused**, and so is one bound to such a
  parameter: a mapping writes values, which would replace the expression for good. `Arm` returns
  False, and clicking the control in map mode says which parameter is driven.
- **A parameter that becomes driven after it was mapped is skipped**, never overwritten, and
  reported by the new `DrivenRows()`. The editor marks it `(driven)`, and `PruneDead` leaves it alone.

### PresetManager 4.3.0 and PresetSnap 1.2.0

Exported preset files now say what wrote them, and imports check it.

- **Every export carries `version`, `tool`, `toolVersion` and `exported`** beside the preset data.
  The schema version answers "what shape is this file"; `toolVersion` answers "which build wrote
  it", which the schema version cannot, since one schema spans many releases.
- **Imports refuse a file from a newer TDXMorph** rather than reading it as far as it happens to
  parse, and say so naming both versions. Nothing is touched when a file is refused. Older files,
  and files written before the stamp existed, import exactly as before.
- **Pointing Import at a file that is not a TDXMorph export now tells you so** instead of failing
  with a raw error.
- PresetSnap's file format is its own and is not interchangeable with PresetManager's. Its version
  number is now named to say so, and the two are free to diverge.

### PresetSnap 1.0.0

First release. Plain store and recall for any COMP, with no morph engine at all.

- **Zero setup.** Drop it into a base or container and a `Presets` page appears on that component.
  Every custom parameter on the host is tracked from the start.
- **It does not interpolate**, which is the point of it. PresetManager moves between states over
  time; this one snaps. Use whichever the moment calls for.
- **Real parameters on the host**, so storing and recalling are MIDI and OSC mappable and work
  with the panel closed. Changing the `Preset` menu recalls, so there is no separate Recall button.
- **A 16 pad grid.** Click STORE to arm, then a pad to capture into it. A filled pad recalls with
  one click.
- **The whole data layer is one table** you can read, hand edit and diff. Rows are parameters,
  columns are presets. No hidden storage, no JSON blob.
- **A recall never overwrites an expression, an export or a bind.** Assigning a value in
  TouchDesigner also forces the parameter into constant mode, so a naive recall would silently
  freeze your expression into a number. The mode is checked live at recall time, so a row starts
  working again by itself once you remove the expression.
- **Capture parameters of operators inside the host** by dragging the operator onto the editor.
- **Presets export as JSON in the PresetManager shape**, so a project that outgrows plain snapping
  can move its presets to the bigger tools.

Note that deleting the component does not remove the page it injected: pulse **Remove Page** first,
or turn `Injectpage` off. An Execute operator has no destroy event, and the one in question lives
inside the thing being deleted.

Full guide: [PresetSnap](Documentation/PresetSnap.md).

### PresetManager 4.2.0

Each preset now carries its own morph time, curve and distribution. Global mode used to borrow the
first element's timing, which meant a preset could not describe how long it should take to arrive.
Existing presets are migrated on load.

### ListView 1.0.6

Right-aligned cell text no longer loses its last character. The horizontal text offset always
displaces to the right, so the padding has to follow the justify edge rather than being applied in
one direction.

### ControlMapper 1.0.2

Picks up the ListView fix above, which its mapping editor renders through. No other change.

## [Open Toolkit 4.1.7] (3rd quarter of 2026)

PresetManager **4.1.7**, ControlMapper **1.0.1**, PresetInspector **1.5.5**, JSONTree **1.5.5**,
ListView **1.0.5**.

Smaller downloads, nothing else.
PresetManager drops from 119.5 KB to 86.6 KB, PresetInspector 21.9 to 13.8, JSONTree 17.9 to
11.9, ListView 26.7 to 18.8 and ControlMapper 46.8 to 30.5. No behaviour, parameter or API
changed in any of them.

## [Open Toolkit 4.1.6] (3rd quarter of 2026)

PresetManager **4.1.6**, ControlMapper **1.0.0**, PresetInspector **1.5.4**, JSONTree **1.5.4**,
ListView **1.0.4**.

ControlMapper joins the free tier as a component in its own right.

### ControlMapper 1.0.0

First release as a standalone component. It has been part of ParameterMorpher and SceneLauncher for
some time; this is the same mapping service, packaged so you can drop it into any project.

- **One service for MIDI and OSC.** Both protocols normalise into a single channel namespace, so a
  mapping does not care which one it came from.
- **Any custom parameter is mappable.** A control needs no special wrapper, only a small marker
  operator that catches the click while map mode is on.
- **Mappings live in a table you can read.** They are not buried in storage, so they can be
  inspected, edited and diffed like anything else.
- **Per-mapping range**, taken from the parameter itself when the mapping is learned, rather than one
  global range shared by everything.
- **Soft takeover.** A control can be set to write only once it crosses where the parameter already
  sits, which removes the jump the first time you touch a fader. The default stays immediate.
- **Dead mappings are shown and can be pruned**, rather than silently skipped.
- **An editor** with inline editing, re-learn, delete and drag-reorder.

### PresetManager 4.1.6, 4.1.5 and 4.1.4

#### Bug fixes

- **The morph curve ships at Linear again.** An ease and a curve coefficient of 0.75 had been
  captured into the released component, so a fresh PresetManager morphed on an Easein curve rather
  than the documented Linear. (4.1.4)
- **A deleted element no longer raises an error every frame.** The engine held a cached parameter
  belonging to an element that had been destroyed and wrote to it without checking it was still
  valid. (4.1.5)
- **The Paths editor's list background follows its row colour.** The empty area of the list, and the
  band below the last row, stayed on a fixed grey when the row colour changed. (4.1.6)

#### Changed

- **The blend amount drives the engine through the cook chain.** Moving the blend used to force a
  cook from Python. The value now propagates by the normal dependency path, and nothing cooks while
  the blend sits still. (4.1.5)

### ListView 1.0.4

#### Bug fixes

- **The list background follows the row colour.** The underlying list operator carried a fixed
  background that happened to match the default row colour, so any list given a different row colour
  kept the old grey in its empty area and below the last row.

## [Open Toolkit 4.1.3] (3rd quarter of 2026)

PresetManager **4.1.3**, PresetInspector **1.5.4**, JSONTree **1.5.4**, ListView **1.0.3**.

The engine was substantially rewritten and the free tier grew from one component to four.
Presets written by older versions migrate automatically. Please report anything you find on the
[issue tracker](https://github.com/DarienBrito/TDXMorph/issues).

### PresetManager

#### New features

- **Multi-track morphing engine.** Every tracked node can morph on its own clock, with its own duration, curve, shape coefficients, end mode, group and distribution. Enabled by the **Multi-Track Engine** toggle on the Options page, and **off by default**, so existing projects keep the 3.2 single-track behaviour unchanged.
- **Preset schema v2.** Timing moved from the top of a preset down into each tracked path, which is what makes per-track timing possible. Presets written by older versions migrate automatically and idempotently, with no data loss.
- **End modes per track:** hold, loop, ping-pong or advance.
- **Per-track auto sequences.** One node can walk its own preset or random cadence while its neighbours do something else.
- **Manual blending.** Load two presets and crossfade them by hand with a single factor, through the existing morph chain, with no clock running.
- **Curve-shape authoring.** Global `Curvea`, `Curveb` and `Curven` plus per-path columns in the paths editor. Coefficients reset to each curve's own defaults when the curve changes.
- **A View column in the paths editor** that opens or reuses one floating network pane on the tracked operator, selects it and frames it.

#### Bug fixes

- **Stop now actually stops.** Stopping left the per-track auto cadences and both deferred queues armed, so a Stop issued while a track was completing was undone a frame later, restarting the clock.
- **Stopping no longer makes an untriggered track replay itself.** Track start times are absolute clock seconds, but stopping zeroed the clock, leaving stored starts in the future of the new epoch. A neighbouring track would freeze and later run a morph nobody asked for.
- **A deleted parameter no longer aborts the whole morph.** A stored parameter the user had since removed took down every other track with it. Missing names are now skipped, with one message per target naming them.
- **The blend menus cannot hijack a running morph.** Changing preset A or B while blending was off still overwrote the values of a morph in progress.
- **Loading a preset no longer rewrites your session defaults.** A loaded preset permanently replaced the session morph time, curve and distribution, so a later random morph ran at the preset's duration and ignored your global setting. Preset timing is now scoped to its own trigger. An explicit `morphTime` or `morphCurve` you pass in is still deliberate and still persists.
- **A morph time you pass in now wins over the times stored in the preset.** With multi-track on, an override was accepted and then ignored because each track fell back to its own stored duration.
- **A morph of zero seconds never finished.** It reached the target and left the engine running and cooking at rest, and the next morph started from stale state.
- **Importing a preset file written by an older version could not be morphed.** Import and inject now migrate the file instead of storing it as-is.
- **Migration no longer drops the curve shape.** A preset carried over from v1 lost its shape coefficients, so an Easein curve reached its target about a quarter of the way through.
- **Importing presets held by another component works.** The validation guarding against malformed payloads rejected TouchDesigner's own stored dictionaries.
- Re-storing a preset no longer flattens its per-track end mode, group and quantize settings.
- A preset sequence with an explicit key list no longer raises an index error.
- Malformed data passed to `InjectPresets` is now rejected on the way in, rather than surfacing much later as an unrelated error.
- Resetting the morph time no longer gives an instant morph. The value and the default were 2 and 0; both are 2.0 now.
- The shipped file no longer carries a leftover development path in its `Paths` storage.
- The morph count no longer ships at 2 with a default of 0.

#### Performance

- **Nothing cooks when idle.** The master clock is now a Timer CHOP that runs only while morphing, replacing an always-dirty Speed CHOP and a force-cook keep-alive. A stopped timer is clean, so the whole chain idles.
- **Cached parameter writeback** and **in-place channel updates**: about 4.6x faster on the hot path (351 µs to 75 µs for 400 channels).

#### Changed

- **The paths editor is now the owned MIT ListView**, replacing the Derivative Paths Lister. **PresetManager carries zero third-party content.** Every column it had is kept.
- Curves are consolidated into `curveLib` (16 functions plus their defaults). The legacy `CurvesGenerator` chain is gone.
- Several methods were removed or renamed. See the [API reference](Documentation/PresetManager.md#removed-since-32) for the full list and its replacements.

### PresetInspector (new to this repository)

The preset viewer and value editor, previously bundled, now ships here as its own MIT component at **1.5.4**. Attach it to a PresetManager to browse every stored preset as a tree, or point it at a JSON file. Values are editable, and write-back is guarded so a refused write is reported rather than silently dropped.

### JSONTree (new)

A reusable collapsible JSON tree viewer and inline editor at **1.5.4**, built on the native `listCOMP`. Search, expand and collapse, type-coloured values, editable leaves, and schema-aware rendering that collapses a TDMorph parameter dict into a single readable row. This is the viewer embedded in PresetInspector, released standalone so it can be used for any JSON inside TouchDesigner.

### ListView (new)

A reusable flat-columnar list widget at **1.0.3**, built on the native `listCOMP`. Declarative column spec, inline editing with type coercion, drag-to-resize columns, drag-to-reorder rows, and a pluggable Callbacks module. It is the widget the PresetManager paths editor and the SceneLauncher lists are built on, released standalone and MIT so it can be used in commercial and closed-source work.

# [Release]
## [3.2] (4th quarter of 2025)


### PresetManager

#### Bug fixes

- Improved help for filtering patterns (right-click the filter column to get suggestions).
- Expanded README help for regular expressions (right-click the filter column and select Help).
- Improved PresetManager callback system and fixed a bug that caused inconsistent triggering.

#### New features

- Added new Inspector utility node with text-based preset editing. Connect it to your PresetManager to fetch and manually overwrite presets as needed.
- Made MorphType property dependable.

### Parameter Morpher

#### Bug fixes

- Fixed dependable dictionary error occurring on save in recent TD versions.
- Streamlined code by removing unnecessary manual table editing options.
- Fixed MIDI and OSC mapping mechanisms.
- Resolved multiple issues introduced by recent TD updates.
- Simplified dropdown menus; codebase is now smaller and cleaner.
- Refactored several components for improved simplicity and compactness.
- Fixed an issue preventing toggles and menus from randomizing in certain cases.
- Fixed an issue where locks for non-interpolatable parameters were ignored when ivoking presets.

#### New features

- Improved Inspector functionality for MIDI and OSC mappings.
- Added Perform Mode to reduce memory usage after preset setup.
- TDMorph now supports all parameter types except Header, Sequence, and Python.
- Scripts can still be launched per preset via Add Script in an ElementsContainer.
- Added randomization and locking for non-interpolatable elements.
- Global and Local modes are now stored independently, enabling separate timelines, cuves, and distributions per preset.
- Int sliders do not try to interpolate anymore, therefore behaving more consistently.

### Scene Launcher

#### Bug fixes

- Fixed numerous issues caused by latest TD updates
- Fixed an issue that prevented scripts from being triggered on "Sequence" action.
- Fiex an issue that computed delay times erronously on "Sequence" action.

#### New features

- Removed transport functionality, as it does not align with the tool's purpose.

# [Alpha release]
## [3.1.1] (3st quarter of 2024)

#### Bug fixes

- Fixed font issue discrepancy on Mac that showed empty text on components. Default is now Verdana and Material Design Icons, which work on both systems.

# [Alpha release]
## [3.1.0] (2st quarter of 2024)

This release was mainly focused in solving things that got broken along the way with TD updates. It has a better implementation of nodes, a cleaner and update UI and an overall smaller memory footprint. So should be lighter and faster.

### PresetManager

#### New features

- Added pulse to "overwrite" current target preset, which speeds up workflow. Before one had to change the name of preset and store. Now this is done in a single click under the "Actions" header.

- Added callback functions to trigger events on completion:
	onMorphingStart()
	onMorphingEnd()
	onPresetCall()

#### Bug fixes

- Timing possibilites were broken in latest TD versions, due to a change in timer's "Active" parameter from a toggle to a menu
- Various issues apperared on cloning with latest version, not sure why precisely... but these have been fixed.
- Fixed a bunch of broken functionality in quantization

### Parameter Morpher

#### Bug fixes

- All PresetManager-related issues

#### Changes

- Re-designed the way UI elements are created
- All code is now moved to a "Modules" base and all classes now reference the code with selects. This reduces file size and is dramatically easier to mantain.
- Updated certain UI elements to use current standards, such as replacement of dropdown menu's and text operators with latest optimized versions.
- Removed a lot of unnecessary parameters and code on low level, which made things lighter and easier to understand for developers.

### Scene Launcher

#### New features

- Added callback functions to trigger events on completion:
	onMorphingStart()
	onMorphingEnd()
	onPresetCall()

#### Changes

- Similar UI and code enhancements as with "Parameter Morpher"

# [Beta release]
## [3.0.0] (1st quarter of 2023)

### PresetManager

#### New features

- New node addition mechanism via UI drag and drop
- Paths can now be updated automatically 
- Paths can be edited manually too, although the automatic method is recommended
- Each node can now have its own wildcard to capture specific parameters
- Capturing wildcard support has been extended to regular expressions
- Added interpolation output to PresetMorpher
- Added blending possibilities on the COMP level

#### Bug fixes

- An inconsistency that prevented adding new parameters after creating some presets
- An error caused by the new Header parameter type
- An inefficiency that made settings jump when interpolating large data-sets
- A redundancy that unnecessarily extra data for each parameter
- An excess of dependable storage has been removed 
- Completion signal for morphings is now reliable.

#### Changes

- Replaced all calls of getattr and setattr, which are slow, with direct dictionary calls
- The input table to specify paths has been removed
- PresetManager now relies on the internal Path component for node paths
- Interpolation curves have been changed to CPU-based, to ensure frame-by-frame synchronicity
- RandomMorph method was renamed to MorphRandom
- Randomize method renamed to SetRandom
- SetPreset method renamed to MorphPreset
- JumpToPreset method renamed to SetPreset
- Saw, Step, Perlin and Simplex interpolations have been removed
- Removed paths specification via a table, which should be done now via de given UI
- Renamed and re-structured various parameters in the Base
- Two new parameters as A and B targets for blending have been added at the Base level
- All extUI extension classes have been removed (better design)

### ParameterMorpher 

#### New features

- Added "Freeze" mode, which disables all interaction for sliders and randomizations to avoid unwanted changes when a set of desired parameters is found.

#### Bug fixes

- Many inefficiencies have been resolved and the UI should feel lighter.
- File size is greatly reduced.
- A lot of other things I did not keep track of :P 

#### Changes

- Replaced all calls of getattr and setattr, which are slow, with direct dictionary calls
- Previously this was the whole library, which now has been rebuilt to be a UI front-end only
- Because of that change this is now called the Parameter Morpher
- Internal path allocation has been completely revamped
- All extUI extension classes have been removed (better design)

#### Workflows

- If you delete all elements in a container and there were presets saved,
if you recreate the elements and use global mode the presets will still 
be there and functional for available elements. To make them work locally
as well resave the presets in global mode one by one and voila.

### Scene Launcher

#### Changes	

- Callbacks system has been refactored to a simpler model
- System-wide refactor of UIs. Should be faster and smoother.


# [Released]

## [2.0.0] (2nd quarter of 2020)

### New features
- Scripts are now possible in ElementsContainer and SceneLauncher
- New node: SceneLauncher, allows for fine detail control for preset launching and sequencing
- Possible to select change on start/end of morphing for parameters that are non-interpolatable
- Possible to use patterns via code (experimental)
- New patterns library created
- Integration with master clocks via the Beat and Ableton Link CHOPs, for LFO's and Patterns per slider
- Blending between two arbitrary parameters (experimental)
- Possible to drag and drop to re-order presets in the UI
- New node: PresetsAnimator, to convert stored presets to Animation COMP automatically
- New node: ExposeChannels, lets you grab all channels for each slider in a SlidersContainer
- Possibility to "Freeze" a given parameter, to avoid editing things by accident
- Two new interpolation modes (snapIn, snapOut)
- Jump to preset with Ctrl+left click (no interpolation)
- Morph to preset with Ctrl+right click (arbitrary "quick check" time, disregard of stored preset time)
- Cancel morphing wiht Ctrl+middle click
- Shift left-click over slider name to reset slider to original default value. 
- Now its possible to Stop/Play/Pause a current morphing in PresetManager
- Now possible to re-bind data using a bindings table
- New Universal "ElementsContainer" can host most parameter types
- All parameter types with corresponding UI's, except for File and Python supported.
- Blending of arbitary presets is now possible.

### Enhancements
- TDMorph is now faster when morphing, with architecture optimizations
- UI architecture and re-design, a great contribution by Roy Gerritsen from y=f(x) lab
- Hovering on preset now displays stored time and curve
- Optimized architecture, warrantied to not cook when idle
- New optional argument in SetPreset(morphTime=[some value]) allows to override stored preset time with arbitrary value
- Clean up and enhancemente of data flow for external signals in sliders
- Shift right-click over slider name to edit it (instead of double clicking)
- Sync is now a momentary button that writes the time value to all sliders, on any mode.
- Enhanced suite of tools to add ad-hoc modules
- TDMorph is now more efficient, with less unnecessary nodes
- Removed redundant comments and renamed functions to make code more readable

## Fixes
- Fixed problem on updating state of preset on opening/closing a project
- Deleted unnecessary code and improved comments for some methods

## [1.1.4] - 2020-7-04

### Added
- New node: PresetGrabber. Allows for accessing presets in tables that update on any preset change
  
### Fixes
- Fixed problem with clamping values in sliders

### Enhancements
- Presets are now an attribute in PresetManager and are dependable
- Presets can be accessed as PresetManager.Presets.getRaw()
- Deleted unnecessary code

## [1.1.3] - 2020-25-03
### Fixes
- Fixed an issue that caused strange parameters in slider on param drop
- Fixed an inconsistecy on index to set/store/delete presets. All begin now from 1
- Fixed an issue that prevented the use of toggles and menus with sliders
- Fixed an issue that altered presets when chaning morphing time while a recall was running

### Enhancements
- Changed logic of a couple elements for cleaner code
- Deleted unnecessary code

## [1.1.2] - 2020-09-03
### Fixes

- Renamed Parameters now persist across export/import 
- LFO mode now follows range on creation and persists when inspector is closed
- Launching presets and other python commands that got broken have been fixed
- Custom morph time issues after TD update are now fixed
- When reimporting from Json, parameters now preserver order
- Renaming TDmorph now causes no errors

## [1.1.0] - 2020-01-03
### Added

- Infrastructure for lfo and patterns support implemented (will not be yet fully released however)
- Fine control for mappings via a table
- Marker to show preset storage within the preset buttons
- Implemented a "pin" button, to collapse menu to just preset control for workflow enhancement
- Now its possible to change range of sliders to any arbitrary set of values
- Non interpolating parameters such as menus will change on start of trigger
- On completion signal for morphings can now be exported by the user
- Clocks for each slider can now be exported by the user
- Support for menu parameters, where range of slider is the number of items in menu
- Refactoring of SliderUI class to use properties (like it should)
- Deletion of unnecessary methods in SliderContainer and SliderUI classes
- Syncing affects also random distribution and interpolation curves

### Enhancements

- Sliders sub menu redesigned and optimized
- Robust system for auto-learn and mapping editing for OSC
- Less space taken by curve viewer
- Now possible to rename parameters by double-clicking on the name
- To save a preset shift-lclick, to delete it shift-rclick on the preset button
- Got rid of unnecesary  buttons for recall and save of presets
- Got rid of global shortcut for Lib. This allows for multiple instances of TDMorph in the same patch
- Binding is now from sliders to parameter instead of the other way around
- Attribute conversion for some methods, making the code more consistent
- Enhanced flow for changing parameters, more clear for developers

## [Released]

## [1.0.0] - 2019-31-12
- First official release
