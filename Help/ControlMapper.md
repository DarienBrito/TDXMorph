# ControlMapper

The MIDI/OSC mapping service. One instance sits inside ParameterMorpher and SceneLauncher and
serves both protocols from a single list of mappings.

Class: `extControlMapper`. Version 1.1.0.

> **MIT licensed.** It ships from this repository as a standalone component, and also
> travels embedded inside the commercial ParameterMorpher and SceneLauncher, which are
> available through [Patreon](https://www.patreon.com/c/darienbrito).
>
> Full reference: [Documentation/ControlMapper.md](../Documentation/ControlMapper.md).

Introduced in ParameterMorpher 4.5.0 and SceneLauncher 4.3.0, replacing the per-widget MIDI and
OSC learn overlays and the separate `Mapper` and `MapperSatellite` classes. Mappings made before
those versions are not carried forward.

A mapping stores a channel, a target operator, a parameter name, an input range, and a takeover
mode. Because the target is a control plus a parameter name rather than a fixed widget, any custom
parameter can be driven, not just the built-in controls.

## Properties

```python
Mappings
```
Every mapping as a dict, in table order.

```python
NumMappings
```
How many mappings are currently stored.

```python
MapMode
```
True while map mode is on.

## Map mode and learning

```python
SetMapMode(state)
```
Enter or leave map mode. Reveals every registered target and cancels any pending arm.

```python
Arm(widget, parName='Value')
```
Arm `widget.par[parName]` so the next incoming channel binds to it. Returns False, and arms nothing,
when the parameter is driven by an expression or an export, directly or through a bind: a mapping
writes a value, which would replace the expression for good.

```python
OnTargetClick(target)
```
Handle a click on a `MapTarget`. Arms its widget, or disarms it if it was already armed. On a driven
parameter it explains why instead of arming, and cancels any control that was armed before.

```python
Map(chan)
```
Bind the armed target to `chan`. Consumes the arm. Returns True when a row was written, and False
when nothing is armed or the armed parameter has since become driven.

```python
Unmap(chan, target, parName='')
```
Remove mappings matching channel and target, and parameter name when given. Returns how many rows
were removed. All three arguments are strings, not operators.

```python
Cancel()
```
Disarm without changing the map.

```python
IsArmed()
```
True while something is armed.

```python
ArmedTarget()
```
The widget currently armed for learning, or None.

```python
ArmRow(rowIndex)
```
Arm an existing mapping for re-learn, so the next channel re-keys that row. The row keeps its range,
takeover and place in the list. A row whose parameter is driven is refused with the same note as a
click.

```python
ArmedRowIndex()
```
The 1-based table row currently armed, or -1.

## Routing

```python
Route(chan, val)
```
Send one normalized channel value, in the range 0 to 1, to every target bound to it. Returns how
many targets were written. A driven parameter is skipped, never overwritten.

```python
Recompile()
```
Force a rebuild of the routing dictionary. The editor calls this after a table edit.

## Editing mappings

```python
SetMappingRange(rowIndex, lo, hi)
```
Set one mapping's target range, by 1-based table row. A learned menu parameter spans 0 to N, not
0 to N-1, because TouchDesigner floors a menu value to an index.

```python
SetMappingTakeover(rowIndex, mode)
```
Set takeover to `'jump'`, which writes immediately, or `'pickup'`, which waits until the incoming
control crosses the parameter's current value before taking over. A parameter outside the mapping's
range is picked up at that end of the control's travel, and a menu at its selected item. Pickup
re-arms whenever something else moves the parameter after the control took over.

```python
ToggleMappingTakeover(rowIndex)
```
Flip one mapping between `jump` and `pickup`. Returns the new value, or an empty string on failure.

```python
SetMappingChannel(rowIndex, chan)
```
Re-key one mapping onto a different channel.

```python
DeleteMappingRow(rowIndex)
```
Delete one mapping by table row.

```python
ReorderMappings(order)
```
Rewrite the table in the given order, as 0-based data indices. This is what backs drag-reorder in
the editor.

## Dead mappings

```python
DeadRows()
```
The 1-based indices of mappings whose target or parameter no longer resolves. Dead mappings are
kept and reported rather than silently skipped, so a renamed control shows as `(missing)` instead
of quietly doing nothing.

```python
PruneDead()
```
Delete every dead mapping. Returns how many went.

```python
DrivenRows()
```
The 1-based indices of mappings whose parameter is driven by an expression or an export, directly
or through a bind. `Route` skips them, the editor marks them `(driven)`, and `PruneDead` leaves them
alone because they still resolve.

## Clearing

```python
ClearMappings()
```
Drop every mapping. This is the model call and does not ask first.

```python
ConfirmClearMappings()
```
Ask before dropping every mapping. Returns True when the map was cleared outright, False when a
confirmation is pending. This is what the Clear pulse and the editor's "Clear all" both route
through.

## Registration

```python
RegisterTargets(targets)
```
Declare which `MapTarget` operators exist. Hosts that build controls at runtime call this on
create and destroy. Returns how many were registered.

```python
ScanTargets()
```
Find targets by tag under `Maproot` and register them. Type filtered on purpose, since an
unfiltered search on a large host is expensive.

## UI level methods

```python
OpenEditor()
```
Open the mapping editor.

```python
OnBlink(val)
```
Blink driver for the armed target. One execute operator serves the whole host.

```python
ApplySkin(name=None)
```
Switch the colour skin. Returns the name that took effect. The mapping list is a ListView clone and owns the colours, so the service only forwards. Returns an empty string when no editor is present, because the service also runs headless.
