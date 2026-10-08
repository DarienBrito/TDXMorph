# Preset Buttons

> **PROPRIETARY. Licensed, not sold.** Part of the commercial ParameterMorpher and
> SceneLauncher components, available through
> [Patreon](https://www.patreon.com/darienbrito), not from this repository.

## Core-level methods

### Promoted

```python
GetPresetsSequence()
```
Gets the order on which the presets are visually arranged.

```python
Reorder(name, source, receiver)
```
Re-ordering is a bit tricky, but I do so by mantaining the buttons order and simply altering the preset name they refer to. "source" is origin "receiver" is target.
