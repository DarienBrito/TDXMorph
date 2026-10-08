# Preset Animator

> **PROPRIETARY. Licensed, not sold.** Part of the commercial ParameterMorpher and
> SceneLauncher components, available through
> [Patreon](https://www.patreon.com/darienbrito), not from this repository.

## Core level methods

### Promoted

```python
CreateAnimation(withData=None)
```
Make an animation with provided data.

```python
ReportResult(msg, title)
```
Launches a TDXMorph-formatted pop up window with the given message and title.

### Private

```python
checkIfFromTDXMorphUI(target_owner, param)
```
We check and acquire the true target if the input comes from sliders, else performance will be greatly impacted by drawing UI from animation and, even worse, controllingof parameters would depend on having a TDXMorph active. We control directly parameters, not UI elements! In this manner users dont' need a TDXMorph to run their animation.

```python
createChannelAlias(chan)
```
Create a name that could be parsed in the animation.

```python
createChannels(target)
```
Gathers unique channel names for the animation.

```python
createKeyFrames(target)
```
Synthesize keyframes from given data.

```python
createParameterGroups()
```
Extract elements per channel.

```python
getAllChannels()
```
Get all channels ordered.

```python
initialize(withData=None)
```
One may supply a synthetic (made up) dictionary from where to create key frames via withData. PresetComposer uses this for instance to create an animation from scenes with arbitrary properties.
