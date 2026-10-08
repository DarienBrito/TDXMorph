# Element UI

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
EnableSignal(v)
```
Enables or disables this element's signal, and builds or tears down its controls with it. Prefer this over writing Enablesignal directly when scripting: a parameter execute DAT does not fire in the frame its element was created, so a create-then-enable would leave a running signal with no controls. Idempotent.

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
Set the slider to the range and value found on creation (original value on dragging of parameter). The range resets on the element, the slider you drag and the signal settings together.

### Pattern selection

Each method selects the element's pattern type and sets that type's values in one call. Values
are kept per type on the element's `Patterndata` parameter, so switching type and back
preserves what you entered. Written to the model, never to the controls, so they work
whether or not the pane is pointed at this element.

```python
SetPseq(sequence=[0.25, 0.5, 1.0])
```
Play the sequence in order, looping.

```python
SetPrand(sequence=[0.25, 0.5, 1.0])
```
Pick from the sequence at random.

```python
SetPxrand(sequence=[0.25, 0.5, 1.0])
```
Pick at random with no immediate repeats.

```python
SetPshuffle(sequence=[0.25, 0.5, 1.0])
```
Shuffle the sequence once, then loop it.

```python
SetPwrand(sequence=[0.25, 0.5, 1.0], weights=None)
```
Pick at random, biased by the weights, one per value. Without weights, every value gets equal odds.

```python
SetPseries(start, step=1, length=10)
```
Arithmetic series. Finite, so the engine wraps it to repeat.

```python
SetPgeom(start, grow=2, length=10)
```
Geometric series. Finite, so the engine wraps it to repeat.

```python
SetPwhite(lo=0.0, hi=1.0)
```
Uniform random values between `lo` and `hi`.

```python
SetPbrown(lo=0.0, hi=1.0, step=0.01)
```
Brownian motion bounded by `lo` and `hi`.

### Private
