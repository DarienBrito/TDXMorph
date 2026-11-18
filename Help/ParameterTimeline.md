# ParameterTimeline

**Coming soon.** Not released yet; these notes may change before release.

Keyed poses on a beat or seconds timeline, baked on edit and played by the built-in transport
(play, pause, stop, loop) or an external clock (a CHOP such as Nexum's `out_transport`, or the
`Position` parameter). Drives the custom parameters of
the COMP in `Target` and any parameter of an operator inside it that you add. A parameter moved from
outside while it plays is overridden until handed back, optionally gliding; Record turns such moves
into keys.

Class: `extParameterTimeline`. Version 1.4.0.

> **PROPRIETARY. Licensed, not sold.** A commercial component governed by the
> ParameterTimeline EULA (see the `LICENSE` operator inside the component). Available
> through [Patreon](https://www.patreon.com/darienbrito), not from this repository.

Full guide: [ParameterTimeline](../Documentation/ParameterTimeline.md).

## Keys

```python
StoreKey(position=None, snap=True)
```
Store the target's current pose as a key at `position` (default: the current position, see
`CurrentPosition()`; on the **Grid** unless `snap` is False) in the active
variant and select it. A key already at that spot is updated. Only the channels moved since the
timeline wrote them (and those not keyed yet) are stored, every channel with **Key All Channels**.
Returns the key id. Raises `ValueError` when the target has nothing to key or nothing moved; the
**Store Key** pulse reports that on Status instead.

```python
UpdateKey(keyId=None)
DeleteKey(keyId=None, allVariants=False)
```
Merge what moved (every channel with **Key All Channels**) into a key's active-variant values, or
remove them; `UpdateKey` raises `ValueError` when nothing moved. Default: the selected key. A key goes once no variant has values left, or at once with `allVariants`.

```python
SetChannelTransition(keyId, channelId, field, value)
MoveChannelValue(keyId, channelId, position, snap=True)
```
One channel's own `curve` or `length` into a key, on top of the key's (`None`, `''` or `'(key)'`
clears it). Move one channel's value from a key to the key at `position`, made when there is none;
a key left with no values goes. Returns the key that holds the value now, None when the key does not
hold that channel.

```python
AddChannelKey(channelId, position, value=None, snap=True)
DuplicateKey(keyId, position, snap=True)
```
Key one channel at `position` in the active variant, at `value` (default: the timeline's value
there, so the curve keeps its shape); a key already at that spot takes it. Copy a key (every
variant, curve, length, cut, per-channel transitions) to `position`; a key already there takes the
copied values. Both select and return the key (the editor's Ctrl+click and Ctrl-drag).

```python
SelectKey(keyId)
StepKey(direction)
```
Select a key and move Position onto it; in Edit mode, also pose the target. `StepKey(1)` and
`StepKey(-1)` select the next or previous key of the active variant from Position.

```python
MoveKey(keyId, position, snap=True)
```
Move a key. It keeps its marker; only the offset changes. Returns False for an unknown key, or when
its marker is not in the Markers table.

```python
SetKeyField(keyId, field, value)
```
Set `marker` (re-anchors, position kept), `offset`, `cut`, `curve` or `length`. Returns False for an
unknown key or field, or a marker missing from the table.

```python
SetKeyValue(keyId, variant, par, value)
```
Set one stored value. The key must already hold that variant. In Edit mode, an edit to the selected
key in the active variant also poses the target.

```python
LoadPose()
```
Pose the target with the timeline's values at Position, even in Edit mode.

## Channels

```python
Channels
AutoChannels()
ApplyChannels()
SetAngleChannels(pattern)
```
`Channels` lists every channel id. `AutoChannels` takes them from the target's custom parameters
that the **Channels** pattern matches (by parameter or group name), while the timeline has none, or
only automatic ones and no key yet; imported channels are never replaced. `ApplyChannels` (what the
**Channels** parameter calls) adds the target parameters the pattern now matches and removes the
automatic channels it no longer matches, with their values; channels added by hand stay, and nothing
changes without a Target. `SetAngleChannels` makes the float channels matching a pattern (`Rx Ry Rz`, `R*`) angles
and the rest linear, and unwraps the stored keys of each channel that turns angle so each takes the
short way from the one before. What the **Angle Channels** parameter calls.

```python
AddChannel(par, group='main', tuplet=None)
AddItems(items)
RemoveChannel(channelId)
RemoveChannels(channelIds)
Relink(channelId, par)
```
Add a number, toggle or menu (not string-menu) parameter of the target or of an operator inside it
(id = the parameter name, or `<operator name>_<par>` inside the target, `_2`, `_3` on a clash; a
parameter that is already a channel returns its id).
`AddItems` takes `Par` and `ParGroup` items, as a drop on the editor does; a group of two or more is one
lane group. `RemoveChannel` / `RemoveChannels` drop channels and their stored values in one edit. `Relink` points a channel at
another parameter, keys kept: a number only to a number, a toggle, menu or integer only to one of
those. `AddChannel` and `Relink` raise `ValueError` for a parameter that cannot be keyed.

## Variants

```python
AddVariant(name, source=None)
RenameVariant(name, newName)
RemoveVariant(name)
```
Make a variant as a copy of `source` (default: the active one), rename one, or drop one with its
values; a key left with no values goes, and the last variant stays. Typing a new name into the
**Variant** parameter calls `AddVariant` from the variant it replaced. Each returns False when it
cannot apply.

## Override, glide, record

```python
BackToTimeline(names=None, glide=True)
Overridden
Touched()
```
A channel whose parameter was moved by something else since the timeline wrote it is overridden and
left alone. `BackToTimeline` hands the named channels (default: all) back; with **Glide** above 0 a
float eases there over that many seconds (integers, toggles and menus snap, as does `glide=False`,
Write Pars off or Edit on).
`Overridden` lists the overridden channels, `Touched()` the ones a Store would key now. A parameter
under an expression or an export is never written.

**Record** (toggle): while on and the clock runs, channels moved from outside are sampled; turning it
off stores them as keys of the active variant holding only those channels, within
**Record Tolerance** of what was played, and they follow again.

## The timeline

```python
Timeline
SetTimeline(timeline)
ImportFile(path)
```
The timeline as a plain dict (a copy), replace it, or load it from a JSON file (either the timeline
dict or `{'timeline': ...}`). Replacing it rebakes and redraws the editor. The dict format is in the
[guide](../Documentation/ParameterTimeline.md#saving-and-loading-timelines).

```python
Rebake()
Evaluate(position)
CurrentPosition()
```
Rebuild the baked table (edits and marker or tempo table changes do this by themselves; a channel
whose keys did not change is reused). The values at a position, one per channel some key holds, in
`Table['names']` order, without writing anything. The current position: the running transport; else
the Position parameter in Edit mode or with Clock Source `transport` or `position`; else the Clock CHOP
(None when it does not resolve).

```python
TogglePlay()
StopTransport()
ToggleLoop()
SetLoop(start, end, free=False, mode='draw')
```
Play or pause, stop (back to the start), loop on or off, or set the loop range on the grid unless
`free` (`mode`: `'draw'` both ends and Loop on, `'start'` / `'end'` that end, `'move'` keeps the length).

```python
SetKeyHold(keyId, channelId, on)
SetKeyVariant(keyId, variant, on)
```
Add a channel to a key in the active variant, at the value the timeline has there, or drop it from the
key (a key left with nothing goes). Put a key in another variant, as a copy of its values in the active
one, or take it out; a key's last variant stays (raises `ValueError`). Each returns False when nothing
changed.

```python
MarkerNames()
CurveNames()
```
The markers the timeline can see, and the transition curves.

## Status

```python
Status
Report(message)
```
`Status` is the current problem, empty when there is none: an edit or bake problem first, else a
clock problem. It is mirrored on the read-only **Status** parameter and the editor's status bar.
`Report` shows a message until the next edit.

## Editor

```python
OpenEditor()
Refresh()
```
Open the editor window (channels, curve graph with overview, key card); redraw it. What the
**Open Editor** pulse calls. A channel row's `×` stops driving it (two clicks, `RemoveChannels`); the
Variant menu adds, renames or removes a variant; parameters dropped on the window become channels, or
relink a missing channel when dropped on its row.

```python
OpenLook()
```
Open the Look window (the editor's palette icon): the `Look` component inside, with **Skin**,
**Apply Skin** and the editor's colours. From a script, `op('Look').ext.extLook.ApplySkin('Mono')`
writes a skin onto those colours and returns how many values changed; a name it does not know gives
Carbon.

## Behaviour worth knowing

- Writes go to the target's parameters only when **Write Pars** is on and **Edit** is off, only for
  values that changed, and never to an overridden or driven channel. The output CHOP `out1` carries
  the timeline's values for every keyed channel.
- A key holds any subset of the channels. Before a channel's first key and after its last, its value
  holds.
- Markers are read by header (`name` plus `position`, `beat`, `beats`, `time` or `seconds`), else by
  the first two columns, so Nexum's `cues` table works as a Markers DAT directly. A key anchored to a
  marker the table no longer has is left out of playback and named on Status.
- Nothing cooks while the clock stands still. The editor graph redraws on edits only.
