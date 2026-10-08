This is the prototype class for all local Elements, which are based on TDXMorphs' Widgets, which are:

1. TDXMorphFloat
1. TDXMorphButton
1. TDXMorphSlider
1. TDXMorphToggle
1. TDXMorphMenu
1. TDXMorphField

Each one of this has slight variations of this prototype class to fit their local types. The properties here defined are mainly used by the OSC and MIDI Mappers. The prototype is defined as follows:

> **PROPRIETARY. Licensed, not sold.** Part of the commercial ParameterMorpher and
> SceneLauncher components, available through
> [Patreon](https://www.patreon.com/darienbrito), not from this repository.

## Core level methods

### Properties

```python
Name (read only)
```
Get the Element's internal name

```python
Value = float
```
Get/Set the Element's value

```python
Range = (float, float)
```
Get/Set the Element's internal range

### Promoted

```python
OffToOn()
```
Off to on script execution.

```python
OnToOff()
```
On to off script execution.

```python
OnValueChange()
```
On value change script execution.
