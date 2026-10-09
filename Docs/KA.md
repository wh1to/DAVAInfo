# WOTB/TB — KA (KeyedArchive)

Parser for the KeyedArchive container format used inside World of
Tanks Blitz / Tanks Blitz .sc2 and .scg files. A KeyedArchive is a
dictionary (key -> VariantType value) that can nest other archives,
arrays, strings and numeric types.

Every SC2 scene body and every SCG polygon group is stored as a
KeyedArchive.

---

## Purpose

SC2 and SCG files are not flat binary structs — they are tagged
key/value trees. KA.py knows how to decode that tree into plain
Python dicts, lists, strings and numbers that the rest of the
add-on can consume.

---

## Errors

| Class | Raised when |
|-------|-------------|
| KAReadError  | Magic string is not "KA", or the version / value type is unknown |
| KAWriteError | Writing is not implemented |

---

## Supported archive versions

| Constant | Value | Meaning |
|----------|-------|---------|
| KA_VERSION_1       | 0x0001 | Inline key/value pairs |
| KA_VERSION_2       | 0x0002 | Fast name (string) table + pairs |
| KA_VERSION_INHERIT | 0x0102 | Descendant archive; reuses the parent's string table |
| KA_VERSION_EMPTY   | 0xFF02 | Empty archive, no body |

---

## Variant types (Types class)

| ID | Name | Stored as |
|----|------|-----------|
| 0  | NONE | nothing |
| 1  | BOOLEAN | int8 (0/1) |
| 2  | INT32 | int32 |
| 3  | FLOAT | float32 |
| 4  | STRING | string-table index (v2) or length+utf8 (v1) |
| 5  | WIDE_STRING | same as STRING |
| 6  | BYTE_ARRAY | int32 length + raw bytes |
| 7  | UINT32 | uint32 |
| 8  | KEYED_ARCHIVE | int32 length + nested KA body |
| 9  | INT64 | int64 |
| 10 | UINT64 | uint64 |
| 11 | VECTOR2 | 2 floats |
| 12 | VECTOR3 | 3 floats |
| 13 | VECTOR4 | 4 floats |
| 14 | MATRIX2 | 4 floats |
| 15 | MATRIX3 | 9 floats |
| 16 | MATRIX4 | 16 floats |
| 17 | COLOR | 4 floats RGBA |
| 18 | FASTNAME | string-table index |
| 19 | AABBOX3 | 6 floats |
| 20 | FILEPATH | string-table index |
| 21 | FLOAT64 | float64 |
| 22 | INT8 | int8 |
| 23 | UINT8 | uint8 |
| 24 | INT16 | int16 |
| 25 | UINT16 | uint16 |
| 27 | ARRAY | int32 count + tagged elements |
| 29 | TRANSFORM | translation + scale + quaternion |

Strings (STRING, WIDE_STRING, FASTNAME, FILEPATH) are stored as a
uint32 index into the fast name table when a string table exists,
otherwise as an inline length-prefixed UTF-8 buffer.

---

## Public functions

| Function | Purpose |
|----------|---------|
| readKA(stream, parentStringTable=None) | Read any KA version, auto-detected from the version field |
| readKA1(stream) | Alias of readKA (kept for backwards compatibility) |
| readKA1Body(stream) | Read a v1 body directly |
| readKA2Body(stream) | Read a v2 body directly |
| readKAInheritBody(stream, parentStringTable) | Read a v0x0102 body using a parent string table |
| readValue(stream, valueType, stringTable) | Decode one VariantType value |
| decodeTypedBlob(blob) | Decode a material-property byte blob: first byte is the VariantType, rest is the packed value |

---

## Archive body layout

### Version 1 (0x0001)

    uint32 count
    repeated count times:
        int8    keyType
        value   key
        int8    valueType
        value   value

### Version 2 (0x0002)

    uint32 fastNameCount
    repeated fastNameCount times:
        int16 length
        utf8  name
    repeated fastNameCount times:
        uint32 stringId       (maps index -> name)
    uint32 count
    repeated count times:
        uint32 keyStringId
        int8   valueType
        value  value

### Version 0x0102 (inherited)

    uint32 count
    repeated count times:
        uint32 keyStringId    (resolved against the parent table)
        int8   valueType
        value  value

### Version 0xFF02 (empty)

    no body

All archives start with:

    bytes "KA"
    uint16 version

---

## Notes

- Nested KEYED_ARCHIVE values pass the current string table down, so
  inner archives may reuse the parent's name list.
- A descendant archive (0x0102) without a parent string table raises
  KAReadError.
- Material "properties" blobs are stored as BYTE_ARRAY; use
  decodeTypedBlob to turn them into a Python value.
- Writing is not implemented (writeKA is a stub).
