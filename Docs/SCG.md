# WOTB/TB — SCG (Scene Geometry Container)

Parser for the .scg file — the geometry container used by World of
Tanks Blitz / Tanks Blitz. An .scg file holds raw vertex and index
buffers grouped by PolygonGroup. It has no idea of hierarchy,
names or materials; those live in the matching .sc2 file.

SCG is a flat, id-keyed bag of geometry.

---

## Purpose

SCG.py turns a raw .scg binary into a Python dict:

    polygonGroupId -> raw KeyedArchive node

The importer then builds one Blender mesh per polygon group and
attaches it to the entity in the .sc2 that references the same id.

---

## Errors

| Class | Raised when |
|-------|-------------|
| SCGReadError  | Magic string is not "SCPG" |
| SCGWriteError | Writing is not implemented |

---

## File layout

    bytes   "SCPG"
    int32   version
    int32   nodeCount
    int32   nodeCount    (duplicate, purpose unknown)
    repeated nodeCount times:
        KA  node

Each KA node is expected to have `##name == "PolygonGroup"`. Nodes
with any other name are skipped with a warning.

The polygon group id is the little-endian integer encoded in the
node's `#id` field.

---

## Public functions

| Function | Purpose |
|----------|---------|
| readSCG(stream) | Parse an .scg stream and return dict {id: KA node} |
| writeSCG(stream) | Not implemented |

---

## Returned structure

    {
        0x1234567890ABCDEF: {
            "##name": "PolygonGroup",
            "#id": b"...",
            "vertexCount": ...,
            "vertexFormat": ...,
            "vertices": b"...",
            "indices": b"...",
            "indexFormat": 0 | 1,
            "indexCount": ...,
            "rhi_primitiveType": 1 | 2 | 10,
            "primitiveCount": ...,
            "textureCoordCount": ...,
            "cubeTextureCoordCount": ...,
        },
        ...
    }

The raw dict is handed straight to PolygonGroup, which knows how to
decode the vertex buffer and index buffer.

---

## Notes

- The second `nodeCount` field is read but ignored. It is believed
  to be a duplicate count, but no file has ever differed from the
  first one.
- A polygon group id is a 64-bit unsigned integer; Python ints are
  used throughout, so no overflow occurs.
- SCG is always used together with an SC2 scene of the same base
  name; loading an SCG alone will produce unnamed meshes.
