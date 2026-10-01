# NonInterpolatableElementUI

> **PROPRIETARY. Licensed, not sold.** Part of the commercial ParameterMorpher
> component, available through [Patreon](https://www.patreon.com/c/darienbrito), not from
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
Change the name of the slider in the custom entry (maintains original name intact).

```python
GetPresetsManager()
```
Returns the deply dependable dictionary that holds presets.

```python
Reorder(name, source, receiver)
```
Reorder widgets.

```python
SetDefaultValue()
```
Set the slider to the range and value found on creation (original value on dragging of parameter).

### Private
