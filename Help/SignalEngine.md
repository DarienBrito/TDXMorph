# Signal Engine

> **PROPRIETARY. Licensed, not sold.** Part of the commercial ParameterMorpher
> component, available through [Patreon](https://www.patreon.com/darienbrito), not from
> this repository.

One service at the component root drives the LFOs and patterns of every element, rather than
each element carrying its own engine. It reads the fifteen signal parameters on each element's
`MorphSettings` operator and writes the result straight back to the mapped parameter.

Only enabled signals occupy the service's table, and its clock runs only while that table has
a row, so a component with no signals enabled does not cook when idle.

You rarely need to call this class. Setting the model parameters is enough: the component
rebuilds the service for you whenever one of them changes. The methods below exist for
scripted rebuilds and for the service's own internals.

## Core-level methods

### Properties

```python
ActiveSignals
```
The signals currently in the table, that is, the elements whose `Enablesignal` is on.

### Promoted

```python
Rebuild()
```
Rewrite the signal table from the live model, parse each signal's settings once and re-cache
the write targets. Called automatically when any of the fifteen signal parameters changes.

```python
Generate(scriptOp)
```
Cook one sample per active signal from the settings parsed at the last `Rebuild`, without reading
the table. Driven by the service clock, so it stops when the table empties.

```python
RefreshPatterns()
```
Rebuild only the cached patterns whose type or values changed, then re-read the model. An
unchanged pattern keeps its position and held value, so redrawing an editor never restarts it.

```python
Write(chanName, val)
```
Write one channel back to its target parameter, using the cached reference rather than
resolving a path every frame.
