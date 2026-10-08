# Patterns

> **MIT licensed**, but distributed only inside the commercial ParameterMorpher
> component, which is available through [Patreon](https://www.patreon.com/darienbrito), not
> from this repository.

## Core level methods

### Promoted

```python
Empty()
```
Pattern placeholder.

```python
Pbrown(lo=0, hi=1, step=0.125, length='inf')
```
Brownian motion generator.

```python
Pfunc(nextFunc, length='inf')
```
Function gets evaluation on call.

```python
Pgeom(start=0, grow=1, length=1)
```
Geometric series generator.

```python
Prand(sequence, repeats=1)
```
Random number generator.

```python
Pseq(sequence, repeats=1, offset=0)
```
Sequencer pattern.

```python
Pseries(start=0, step=1, length=1)
```
Arithmetic series generator.

```python
Pshuf(sequence, repeats=1)
```
Shuffle given sequence once.

```python
Pwhite(lo=0, hi=1, length='inf')
```
White noise random generator.

```python
Pwrand(sequence, weights, repeats=1)
```
Random generator based on given weights.

```python
Pxrand(sequence, repeats=1)
```
Random generator with no consequtive repeats.
