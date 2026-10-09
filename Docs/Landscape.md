# WOTB/TB — Landscape (Heightmap)

Reader and mesh builder for DAVA terrain. World of Tanks Blitz /
Tanks Blitz stores each map's terrain as a .heightmap file (a grid
of uint16 heights) plus an optional .bbox file (the world-space
bounding box the grid maps onto).

This module reads the heightmap, un-tiles it if needed, and builds
a triangulated grid mesh that Blender can display.

---

## Purpose

Maps in WoTB/TB are not meshes — they are heightfields. The
importer uses Landscape.py to reconstruct the terrain as a regular
grid of vertices whose Z comes from the heightmap and whose X/Y
come from the bounding box.

---

## Errors

| Class | Raised when |
|-------|-------------|
| HeightmapReadError | File is too small or the size cannot be determined |

---

## Constants

| Name | Value | Meaning |
|------|-------|---------|
| MAX_HEIGHT_VALUE | 65535.0 | uint16 heights map linearly onto the bbox Z range |

Heights are normalised: `z = minZ + (h / 65535) * (maxZ - minZ)`.

---

## Heightmap file layout

    uint32  size        (grid side length, grid is size x size)
    uint32  blockSize   (tile side length; 0 or 1 means untiled)
    uint16  heights[size * size]   (stored in blockSize x blockSize tiles)

Total file size must be `8 + size * size * 2` bytes. If the header
does not match, the reader falls back to deriving a square size
from the remaining bytes.

---

## Tiling

DAVA stores heightmaps in square tiles to improve cache locality.
The on-disk order is:

- blocks in row-major order,
- pixels inside a block in row-major order.

`_detile(raw, size, blockSize)` rearranges them into a flat
row-major grid.

If `blockSize <= 1`, or `blockSize >= size`, or `size` is not
divisible by `blockSize`, the data is returned unchanged.

---

## Public functions

| Function | Purpose |
|----------|---------|
| readHeightmap(path) | Returns `(size, heights)` with heights flattened row-major |
| buildLandscapeMesh(size, heights, bbox, step=1, flipV=True) | Returns `(verts, faces, uvs)` |
| parseBBox(blob) | Unpacks a 24-byte blob of 6 floats into `(minX,minY,minZ,maxX,maxY,maxZ)` |

### readHeightmap

1. Reads the whole file.
2. Validates `size` and total length.
3. Falls back to `isqrt((len-8)//2)` if the header looks wrong.
4. Unpacks `size * size` uint16 values.
5. De-tiles with `blockSize`.
6. Returns `(size, heights)`.

### buildLandscapeMesh

Arguments:

| Arg | Meaning |
|-----|---------|
| size | Grid side length |
| heights | Flat row-major list of uint16 |
| bbox | (minX,minY,minZ,maxX,maxY,maxZ) |
| step | Sampling step, >= 1. Larger = fewer vertices |
| flipV | Flip the V coordinate (Blender uses bottom-left origin) |

Behaviour:

- Samples the grid at indices `0, step, 2*step, ...` and always
  includes the last row/column so the mesh edges align with the
  bbox edges.
- Produces `verts` as (x, y, z) tuples in world space.
- Produces `uvs` as (u, v) per vertex, normalised to 0..1.
- Produces `faces` as quads, not triangles.

Returns `(verts, faces, uvs)`.

### parseBBox

Landscape bbox is stored as 6 little-endian float32 values:
`minX, minY, minZ, maxX, maxY, maxZ`.

If the blob is missing or shorter than 24 bytes, a unit box
`(-1,-1,0, 1,1,1)` is returned.

---

## Notes

- The heightmap is always square. Rectangular grids are not
  supported.
- Heights are uint16; the actual vertical scale comes entirely from
  the bbox. A heightmap without a bbox produces a 1-unit-tall
  terrain.
- `step` is a cheap LOD control: `step=1` imports every vertex,
  `step=4` imports one in sixteen. The add-on exposes this as
  "Landscape step" in the import dialog.
- Used by: ImportDAVA (via DAVASceneBuilder) and ImportLandscape.
