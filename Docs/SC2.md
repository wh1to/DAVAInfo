# WOTB/TB — SC2 (Scene Descriptor)

Parser for the .sc2 file — the scene descriptor used by World of
Tanks Blitz / Tanks Blitz. An .sc2 file describes the entity
hierarchy of a vehicle or map (nodes, transforms, render
components, materials, animations) but contains no geometry.

Geometry lives in the matching .scg file.

---

## Purpose

SC2.py converts a raw .sc2 binary into a SC2Scene object that the
importer can walk: it exposes the hierarchy tree, the material
table, the animation table and lookup helpers by id or by name.

---

## Errors

| Class | Raised when |
|-------|-------------|
| SC2ReadError  | Magic string is not "SFV2", descriptor size is negative, body is not a KA dict |
| SC2WriteError | Writing is not implemented |

---

## File layout

    bytes   "SFV2"
    int32   version
    int32   nodeCount
    KA      (pre-header archive)
    int32   descriptorSize
    bytes   descriptor[descriptorSize]
    KA      body

The body is a KeyedArchive with three top-level keys:

| Key | Type | Contains |
|-----|------|----------|
| #dataNodes | array | Flat list of every data node (materials, animations, ...) |
| #hierarchy | array | Nested entity tree (scene graph roots) |
| #sceneComponents | dict | Scene-wide components |

---

## SC2Scene class

Built by readSC2. Holds the parsed data plus several indexes.

### Attributes

| Attribute | Meaning |
|-----------|---------|
| version | SC2 format version from the header |
| dataNodes | Flat list of data nodes |
| hierarchy | Nested list of root entities |
| sceneComponents | Scene-wide components |
| materialsById | id -> material node |
| animationsById | id -> animation node |
| entitiesById | id -> entity dict |
| entitiesByName | name -> entity dict |

### Indexing

`_build_indexes` scans `dataNodes` for entries whose `##name` is
`nmaterial` or looks like an animation node, then walks the whole
`hierarchy` collecting every entity by id and by name.

### Animation node detection

`_is_animation_node` returns True when:

- `##name` is `animationdata` or `animation`, or
- the node has a `keyCount` field and any `key_*` field.

### Entity helpers

| Method | Returns |
|--------|---------|
| get_entity_id(entity) | id from `#id` / `id` / `entityId` |
| get_entity_name(entity) | name from `name` / `#name` / `entityName` / `nodeName` |
| get_children(entity) | list from `#hierarchy` / `hierarchy` / `children` |
| iterEntities() | Generator over every entity in the tree |
| findEntityById(id) | Entity by id |
| findEntityByName(name) | Entity by name |

### Animation helpers

| Method | Returns |
|--------|---------|
| findAnimationComponent(entity) | The AnimationComponent dict attached to an entity |
| getEntityAnimationId(entity) | animation id referenced by an entity |
| getEntityAnimation(entity) | The full animation node |
| getAnimatedEntities() | List of (entity, animationId, animation) for every animated entity |

Component lookup accepts three shapes:
- top-level `*animationcomponent*` key,
- a `components` / `component` dict,
- a `components` list of dicts with a `type` / `class` / `name` field.

---

## Public functions

| Function | Purpose |
|----------|---------|
| readSC2(stream) | Parse an .sc2 stream and return an SC2Scene |
| writeSC2(stream) | Not implemented — raises SC2WriteError |
| idToInt(value) | Coerce a DAVA id (bytes, int, float, str, hex, dict) into a Python int |

### idToInt

Handles:
- `bytes` / `bytearray` -> little-endian unsigned int,
- `bool` -> 0/1,
- `int` / `float` -> truncated int,
- `str` -> decimal, `0x` hex, or hex-bytes,
- `dict` -> looks at `__bytes_hex__`, `bytes_hex`, `hex`, `$hex`, `value`, `$binary`.

Returns `None` on failure.

---

## Notes

- The body must be a KeyedArchive dict; anything else raises
  SC2ReadError.
- `_import_embedded_animations` is a helper that tries to delegate
  to ANIM.import_sc2_animations if that module is available.
- SC2 contains no mesh data. Pair it with the .scg of the same base
  name to get geometry.
