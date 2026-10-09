# WOTB/TB — StreamBuffer

Binary IO helper used by every parser in the DAVA SC2/SCG Importer.
It wraps a raw file-like object and provides little-endian typed
readers/writers for ints, floats, vectors, matrices, colors, strings
and raw bytes.

All World of Tanks Blitz / Tanks Blitz binary containers
(.sc2, .scg, .anim, .heightmap) are read through this class.

---

## Purpose

DAVA files are binary. StreamBuffer gives the rest of the add-on one
consistent API to read them without repeating `struct.unpack` calls
everywhere.

---

## Constructor

    StreamBuffer(stream, endian="little")

| Argument | Meaning |
|----------|---------|
| stream   | Any object with read/write/seek/tell (open file, BytesIO) |
| endian   | Byte order for integers. DAVA files are always "little" |

---

## Position control

| Method | Description |
|--------|-------------|
| tell() | Current offset in the stream |
| seek(offset, mode=0) | Move cursor |
| skip(count) | Advance `count` bytes forward |

---

## Integer readers

| Method | Size | Signed control |
|--------|------|----------------|
| readInt8(signed=True)  | 1 byte | yes |
| readInt16(signed=True) | 2 bytes | yes |
| readInt32(signed=True) | 4 bytes | yes |
| readInt64(signed=True) | 8 bytes | yes |

## Integer writers

| Method | Size |
|--------|------|
| writeInt8(value)  | 1 byte |
| writeInt16(value) | 2 bytes |
| writeInt32(value) | 4 bytes |
| writeInt64(value) | 8 bytes |

---

## Floating point

| Method | Reads |
|--------|-------|
| readFloat()  | one float32 |
| readDouble() | one float64 |
| writeFloat(v)  | one float32 |
| writeDouble(v) | one float64 |

---

## Composite numeric types

All returned as Python tuples of little-endian float32.

| Method | Returns |
|--------|---------|
| readFloats(count) | list[float] of length count |
| readVector2() | (x, y) |
| readVector3() | (x, y, z) |
| readVector4() | (x, y, z, w) |
| readMatrix2() | 4 floats |
| readMatrix3() | 9 floats |
| readMatrix4() | 16 floats |
| readColor() | 4 floats (RGBA) |
| readAABBox3() | 6 floats: min(x,y,z), max(x,y,z) |

---

## Strings

| Method | Behaviour |
|--------|-----------|
| readString(count) | Reads `count` bytes, decodes UTF-8 with errors="replace" |
| writeString(value) | Encodes UTF-8, writes raw bytes (no length prefix) |

Note: writeString does NOT prepend a length. Callers that need a
length-prefixed string write the length themselves.

---

## Raw bytes

| Method | Behaviour |
|--------|-----------|
| readBytes(count) | Returns bytes |
| writeBytes(value) | Writes bytes |

---

## Notes

- Always instantiate with a byte stream. Text-mode files will break
  readInt8/readFloat/readBytes.
- All integer helpers default to little-endian regardless of the
  endian argument being passed as "little" (hard-coded in the
  unpack format strings for floats and composites).
