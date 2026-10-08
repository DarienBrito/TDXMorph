# NonInterpolatableElementUI

> **PROPRIETARY. Licensed, not sold.** Part of the commercial ParameterMorpher
> component, available through [Patreon](https://www.patreon.com/darienbrito), not from
> this repository.

## Core-level methods

### Promoted

```python
DestroyElement()
```
Destroys the Widget and removes its stored values from every preset.

```python
EditCustomName(name)
```
Sets the element's display name and keeps it through a rebuild. The bound parameter keeps its own name.

```python
GetPresetsManager()
```
Returns the container's PresetManager.

```python
Reorder(name, source, receiver)
```
Reorder widgets.

```python
SetDefaultValue()
```
Set the element to the range and value found on creation (original value on dragging of parameter). A Menu or StrMenu element keeps one slot per item.

### Private
