# WOTB/TB — PolygonGroup

Decoder for a single PolygonGroup node inside an .scg file. A
PolygonGroup holds one vertex buffer plus one index buffer, packed
in a DAVA-specific vertex format.

This module turns the raw bytes into usable Python lists of
positions, normals, UVs, tangents, bone weights and faces that the
importer can hand to Blender.

---

## Purpose

Every mesh in World of Tanks Blitz / Tanks Blitz is a PolygonGroup.
PolygonGroup.py is the layer that understands what the vertex
buffer actually contains: which attributes are present, where they
sit inside each vertex, and how the index buffer forms triangles.

---

## VertexTypes flags

A bitmask describing which attributes a vertex has. The
`vertexFormat` field of a PolygonGroup node is an OR of these bits.

| Flag | Bit | Meaning |
|------|-----|---------|
| VERTEX          | 1<<0  | Position (3 floats) |
| NORMAL          | 1<<1  | Normal (3 floats) |
| COLOR           | 1<<2  | Color (4 bytes BGRA) |
| TEXCOORD0       | 1<<3  | UV set 0 (2 floats) |
| TEXCOORD1       | 1<<4  | UV set 1 |
| TEXCOORD2       | 1<<5  | UV set 2 |
| TEXCOORD3       | 1<<6  | UV set 3 |
| TANGENT         | 1<<7  | Tangent (3 floats) |
| BINORMAL        | 1<<8  | Binormal (3 floats) |
| HARD_JOINTINDEX | 1<<9  | Armor zone index (1 float) |
| PIVOT4          | 1<<10 | 4 floats |
| FLEXIBILITY     | 1<<12 | 1 byte |
| ANGLE_SIN_COS   | 1<<13 | 2 floats |
| JOINTINDEX      | 1<<14 | 4 uint32 bone indices |
| JOINTWEIGHT     | 1<<15 | 4 float bone weights |
| CUBETEXCOORD0..3| 1<<16..19 | Cube map UVs (3 floats each) |

---

## Attribute layout

`_ATTRIBUTE_LAYOUT` defines the packing order and byte size of every
attribute. It is used both to compute the stride and to find the
offset of each attribute inside a vertex.

| Name | Bit | Size (bytes) |
|------|-----|--------------|
| VERTEX          | VERTEX          | 12 |
| NORMAL          | NORMAL          | 12 |
| COLOR           | COLOR           | 4  |
| TEXCOORD0       | TEXCOORD0       | 8  |
| TEXCOORD1       | TEXCOORD1       | 8  |
| TEXCOORD2       | TEXCOORD2       | 8  |
| TEXCOORD3       | TEXCOORD3       | 8  |
| TANGENT         | TANGENT         | 12 |
| BINORMAL        | BINORMAL        | 12 |
| HARD_JOINTINDEX | HARD_JOINTINDEX | 4  |
| CUBETEXCOORD0..3| CUBETEXCOORD0..3| 12 each |
| PIVOT4          | PIVOT4          | 16 |
| FLEXIBILITY     | FLEXIBILITY     | 1  |
| ANGLE_SIN_COS   | ANGLE_SIN_COS   | 8  |
| JOINTINDEX      | JOINTINDEX      | 16 |
| JOINTWEIGHT     | JOINTWEIGHT     | 16 |

Attributes are packed in this order. The stride is the sum of the
sizes of the attributes that are present.

---

## VertexFormat class

Built from the `vertexFormat` integer.

| Attribute | Meaning |
|-----------|---------|
| fmt | The raw bitmask |
| offsets | dict: attribute name -> byte offset, or -1 if absent |
| stride | Total bytes per vertex |

Methods:

| Method | Returns |
|--------|---------|
| has(name) | True if the attribute is present |
| offset(name) | Byte offset inside a vertex, or -1 |

---

## PrimitiveTypes

| Constant | Value | Meaning |
|----------|-------|---------|
| TRIANGLELIST  | 1  | Index buffer is a list of triangles |
| TRIANGLESTRIP | 2  | Index buffer is a triangle strip |
| LINELIST      | 10 | Index buffer is a list of lines |

---

## PolygonGroup class

Constructor takes the raw KA node from SCG.py and decodes
everything.

### Input fields used

| Field | Meaning |
|-------|---------|
| #id | Polygon group id (uint64, little-endian) |
| vertexFormat | Bitmask of present attributes |
| vertexCount | Number of vertices |
| vertices | Raw vertex buffer bytes |
| indices | Raw index buffer bytes |
| indexFormat | 0 = uint16, 1 = uint32 |
| indexCount | Number of indices |
| rhi_primitiveType | One of PrimitiveTypes |
| primitiveCount | Number of primitives |
| textureCoordCount | Number of UV sets |
| cubeTextureCoordCount | Number of cube UV sets |

### Decoded attributes

| Attribute | Type | Notes |
|-----------|------|-------|
| vertices | list[(x,y,z)] | |
| normals | list[(x,y,z)] | |
| colors | list[(r,g,b,a)] | Normalised to 0..1 |
| uvs | list[list[(u,v)]] | One list per UV set |
| tangents | list[(x,y,z)] | |
| binormals | list[(x,y,z)] | |
| hardJointIndices | list[float] | **Armor zone index, stored as float** |
| jointIndices | list[(i0,i1,i2,i3)] | uint32 |
| jointWeights | list[(w0,w1,w2,w3)] | float |
| indices | list[int] | Decoded from uint16 or uint32 |

---

## Face extraction

`getFaces()` dispatches on `primitiveType`:

| Method | Returns |
|--------|---------|
| getTriangleList()  | List of (a,b,c) triples, straight from the index buffer |
| getTriangleStrip() | List of (a,b,c) triples, with strip winding fixed |
| getLineList()      | List of (a,b) pairs |

`getFaces()` returns `(faces, edges)` — one of the two is always
empty.

---

## Notes

- The vertex buffer is always little-endian.
- HARD_JOINTINDEX is a single float per vertex, not an integer. It
  encodes the armor zone used by the overlay builder, not a bone.
- JOINTINDEX and JOINTWEIGHT are the actual skinning data used by
  ANIM/SKELETON to bind a mesh to an armature.
- Colors are BGRA on disk and are converted to RGBA in 0..1 range.
- UV sets are stored in order of the TEXCOORD bits, not by their
  numeric suffix.
