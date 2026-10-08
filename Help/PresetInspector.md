# Preset Inspector

A viewer and value editor for preset data. Attaches to a PresetManager, or reads any JSON
file. A thin wrapper around an embedded [JSONTree](JSONTree.md).

Version 1.5.9. MIT. Capitalized methods are promoted and are the supported API.

Full reference: [Documentation/PresetInspector.md](../Documentation/PresetInspector.md).

## Core level methods

### Properties

These are custom parameters rather than Python properties.

```python
Mode = str
```
The data source: `Preset manager` or `JSON file`.

```python
Presetmanager = OP
```
The PresetManager to read in Preset manager mode. Empty by default.

```python
Json = str
```
The JSON file to read in JSON file mode. The file input is bypassed while this is empty.

```python
Editable = bool
```
Allow editing values. Mirrored onto the embedded tree.

```python
Search = str
```
Filter the tree. Forwarded to the tree's own filter.

```python
Skin = str
```
Colour skin: Carbon, Verdigris or Mono. Handed down to the embedded tree.

### Promoted

```python
ApplySkin(name=None)
```
Switch the colour skin. Returns the name the embedded tree reports back. The Inspector owns no colours itself: it sets the tree's Skin and hands the call down.

```python
WriteBack(keys, newVal, oldVal=None)
```
Persist a tree edit to the active source. `keys` is the key chain from the root of the data to the leaf. Called by the embedded tree on a committed edit, so you rarely call it yourself.

Values only: you cannot add, rename or delete keys. A successful write clears the error
record and re-cooks the source so the tree rebuilds from the committed data.

## Write-back guards

A refused write is recorded rather than silently dropped. Read the reason with:

```python
op('PresetInspector').fetch('lastWriteError', '')
```

| Condition | Recorded reason |
|---|---|
| The Presetmanager parameter does not resolve | `PresetManager not resolved` |
| The PresetManager has its Lock on | `PresetManager is locked` |
| The edited key path is not in the presets | `path not found in presets` |
| The JSON file cannot be read | `cannot read JSON file` |
| The JSON file cannot be written | `cannot write JSON file` |
| The edited key path is not in the file | `path not found in JSON file` |

A successful write sets it to an empty string.

Note that locking the PresetManager also locks the Inspector out of it, which is what you
want during a performance.
