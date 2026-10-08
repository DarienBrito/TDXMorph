# Parameter Morpher

The root of the ParameterMorpher component. Manages the internal library and the tools built
from it.

Class: `extParameterMorpher`. Version 4.18.4.

> **PROPRIETARY. Licensed, not sold.** A commercial component governed by the
> ParameterMorpher EULA (see the `LICENSE` operator inside the component). Available through
> [Patreon](https://www.patreon.com/darienbrito), not from this repository.

This file was previously named `TDXMorph.md`, from the era when the whole toolkit was one
node. It has always documented the ParameterMorpher root.

For the element containers, which is where most of the day-to-day API lives, see
[Documentation/ParameterMorpher.md](../Documentation/ParameterMorpher.md).

## Core Methods

### Public

```python
CreateContainer()
```
Creates an empty container for elements in the ParameterMorpher node.

```python
CreateFromLibrary(item, x=250, y=150, viewer=False)
```
Copies "item" out of the internal `Lib/` at the given coordinates. Enables the viewer if viewer=True. The items are:
  * 'PresetGrabber'
  * 'PresetAnimator'
  * 'MorphingTimers'
  * 'MorphingChannels'
  * 'Patterns'

For a new container use `CreateContainer()`, and for a preset engine of your own use the standalone
[PresetManager](../Documentation/PresetManager.md). Everything else under `Lib/` is internal structure (widget masters, element templates, the look and
module libraries) rather than a library item. Copying one is unsupported and may break on any update.

```python
ExposeChannels(x=0, y=0)
```
Creates a node containing a par CHOP per container, with one channel per element (its value), named `<container>_<parameter>`.

```python
ExposeTimers(x=0, y=0)
```
Creates a node containing select CHOPs for the inner timers of each container.

```python
GetContainer(n)
```
Get the container at n position.

```python
RegisterMapTargets()
```
Tell the ControlMapper which controls are mappable, and return how many were registered. A control is mappable when it carries a `MapTarget` child. The `Lib/` templates are skipped, since they are what `CreateContainer()` and `CreateElement()` copy and a user can never click one, and any widget with `Ignoreprotocols` on is left out, which is how the header chrome stays out of the map. Called on init and whenever the widget set changes, so an element created after load is mappable without a reload. On a component with no container yet this correctly returns 0.

```python
SetMapMode(state)
```
Enter or leave map mode, revealing a click target on every mappable control. Re-registers first, so a control added since load still gets a target. This is what the header **Map** icon drives.

```python
SelectObject(name, x=250, y=0)
```
A convenience method to create items from the library at default locations. This is what the component calls when a new item is invoked.

### Private

```python
containersExist()
```
True if there are containers, false if not.

## UI Methods

### Public

```python
ChangeOrientation(horizontal=False)
```
Changes the layout from vertical to horizontal. Only useful when there is more than one container in the window.

## Custom parameters

| Page | Parameters |
|---|---|
| Dimensions | `Width`, `Height`, `Horizontal` |
| Inputs | `Midi`, `Osc`, `Transport` (CHOP references) |
| Mappings | `Managemappings`, `Clearmappings` (both protocols share one list since 4.5.0) |
| Info | `Containers` (read only) |
| About | The `Readme`, `Help`, `Support` (labelled Patreon), `Website` pulses, then `Author` and `Version`, which are read only |

## Removed

Retired in 4.5.0 with the mapping rebuild: `Managemidimappings`, `Manageoscmappings`,
`Clearmidimappings`, `Clearoscmappings`, `Togglelocationmidi`, `Togglelocationosc`, and the two
per-protocol section headers on the Mappings page. Mappings made in 4.x are not carried forward.

`createBase` is gone. `'SceneLauncher'` is no longer a `CreateFromLibrary` item: it is a
separate product with its own component.

`ReportResult(msg, title)` is promoted here, `op('ParameterMorpher').ReportResult(msg, title)`, and
on each container.
