# Roblox binary file format
This document describes the binary format for Roblox place and model files.
Common extensions for this format include `rbxl` (place) and `rbxm` (model). For
brevity, this document will refer to the format as **RBXL**.

The format is [backward-compatible](#user-content-legacy-format) with the [XML
format][RBXLX], which will be referred to as **RBXLX**.

[RBXLX]: rbxlx.md

# Acknowledgments
This document is based largely on the reverse-engineering efforts of Gregory
Comer ("Havemeat"), author of the [RobloxFileSpec][RobloxFileSpec]. Additional
resources include:

- [rbx-dom](https://github.com/rojo-rbx/rbx-dom)
- [Roblox-File-Format](https://github.com/MaximumADHD/Roblox-File-Format)

[RobloxFileSpec]: https://www.classy-studios.com/Downloads/RobloxFileSpec.pdf

<!--
# Conventions Used in This Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [BCP 14][BCP14]
([RFC2119][RFC2119], [RFC8174][RFC8174]) when, and only when, they appear in all
capitals, as shown here.

[BCP14]: https://tools.ietf.org/html/bcp14
[RFC2119]: https://tools.ietf.org/html/rfc2119
[RFC8174]: https://tools.ietf.org/html/rfc8174

<TODO q="Implement this."/>

-->

# Preliminary concepts

## Type notation
[types]: #user-content-type-notation

The following primitive types are defined for use throughout this document:

Type       | Example     | Description
-----------|-------------|------------
`intNE`    | `int32`     | A signed integer with a size of `N` bits, and [endianness][Endianness] `E`.
`zintNE`   | `zint32b`   | Same as `intNE`, but with [zigzag encoding][ZigzagEncoding].
`uintNE`   | `uint64b`   | An unsigned integer with a size of `N` bits, and [endianness][Endianness] `E`.
`floatNE`  | `float32`   | A IEEE 754 floating-point number with a size of `N` bits, and [endianness][Endianness] `E`.
`rfloatNE` | `rfloat32`  | Same as `floatNE`, but with [rotated encoding][RotatedEncoding].
`bool`     | `bool`      | A boolean value stored as a `uint8`, where 0 is false and 1 is true.
`[N]T`     | `[4]uint8`  | An array with a constant length of `N`, with elements of type `T`.
`[]T`      | `[]uint8`   | An array with elements of type `T`, the length of which is determined dynamically.
`?T`       | `?uint8`    | A value of type `T`, which may or may not be present based on some condition. If not present, the value is omitted entirely.
`T~N`      | `[]int32~4` | A value of type `T`, which is encoded by [interleaving][Interleaving] with a byte size of `N`.
`...`      | `...`       | The type is determined dynamically.

### Endianness
[Endianness]: #user-content-endianness

Types with the `E` suffix may be encoded in either "little-endian" or
"big-endian". If the suffix is `b`, then big-endian is used. Otherwise, the
suffix is empty, and little-endian is used.

With little-endian, the least-significant **byte** is stored first, while with
big-endian, the most-significant byte is stored first.

For example, the integer 305419896 can be encoded in the follow ways:

| Type      | Bytes (hex)   |
|-----------|---------------|
| `uint32`  | `78 56 34 12` |
| `uint32b` | `12 34 56 78` |

### Structures
[structures]: #user-content-structures

A "structure" type comprises a number of ordered fields, each having a certain
type. This document describes structures by using a table with Field, Type and
Description columns. The Field column contains a name that identifies the field.
The Type column contains a type as described above. The Description column
contains a description of the field. Each row of the table is a specific field.
When decoding or encoding, each row is read or written in order, top to bottom.
Fields of type `?T` that are not present are omitted entirely. For example:

Field  | Type     | Description
-------|----------|------------
First  | `int32`  | The first field, having a type of `int32`.
Second | `bool`   | The second field, having a type of `bool`.
Third  | `?int32` | The third field, having a type of `int32`, but is omitted based on some condition. For example, if field Second is false.

### Arrays
[arrays]: #user-content-arrays

When decoding or encoding an array, each element is read or written in order.
For example, to encode value `A` of type `[2]Vector3`, each component would be
written in the following order:

```
A[0].X
A[0].Y
A[0].Z
A[1].X
A[1].Y
A[1].Z
```

### Interpreting types
Consider the type `[5]uint32b~4`. It can be read in the following way:

Section  | Description
---------|------------
`[5]`    | The value is an array of length 5, each element having the following type.
`uint32` | The element type is an unsigned integer of 32 bits in size.
`b`      | The element type is encoded in big-endian.
`~4`     | The bytes of the encoded array are interleaved with a byte size of 4.

## Strings
[string]: #user-content-strings

The **string** type is defined to describe a variable sequence of bytes. It has
the following structure:

Field  | Type      | Description
-------|-----------|------------
Length | `uint32`  | The length of the string.
Bytes  | `[]uint8` | The content of the string. The length is determined by the Length field.

## References
[References]: #user-content-references

The **References** type is defined as `[]zint32b~4` to describe an array of
references. Before encoding to bytes, values in the array are
difference-encoded. The following pseudo-code describes an algorithm for
encoding and decoding:

```
function EncodeDifference(a Array) {
	for i = length(a)-1; i >= 1; i-- {
		a[i] = a[i] - a[i-1]
	}
}
function DecodeDifference(a Array) {
	for i = 1; i < length(a); i++ {
		a[i] = a[i] + a[i-1]
	}
}
```

## Encoding techniques
When encoding and decoding, there are several kinds of transformations that can
be applied to values.

### Zigzag encoding
[ZigzagEncoding]: #user-content-zigzag-encoding

Certain signed integer values are encoded with [Zigzag][zigzag] encoding to
improve compression. The following pseudo-code describes an algorithm for
encoding and decoding:

```
// AND: Bitwise and.
// XOR: Bitwise xor.
// LSHIFT: Bitwise left-shift.
// RSHIFT: Bitwise right-shift.
// K: Size of integer type.
K := 32
K1 := K - 1
function EncodeZigzag(n: intK): uintK {
	a := LSHIFT(n, 1)
	b := RSHIFT(n, K1)
	c := XOR(a, b) as uintK
	return c
}
function DecodeZigzag(n: uintK): intK {
	a := LSHIFT(n, 1)
	b := AND(n, 1) as intK
	c := LSHIFT(b, K1)
	d := RSHIFT(c, K1) as uintK
	e := XOR(a, d) as intK
	return e
}
```

[zigzag]: https://en.wikipedia.org/wiki/Variable-length_quantity#Zigzag_encoding

### Rotated encoding
[RotatedEncoding]: #user-content-rotated-encoding

Certain float values are encoded by applying a circular shift to the bits of the
value one bit to the left, such that the sign bit becomes the least significant
bit instead of the most.

```
S := Sign bit
E := Exponent bits
M := Mantissa bits
MSB := Most-significant bits
LSB := Least-significant bits

SEEEEEEE EMMMMMMM MMMMMMMM MMMMMMMM
               <----
EEEEEEEE MMMMMMMM MMMMMMMM MMMMMMMS
MSB                             LSB
```

Decoding such a float is simply the reverse: apply a circular shift one bit to
the right.

### Byte interleaving
[Interleaving]: #user-content-byte-interleaving

After encoding, the produced bytes of certain value types are **interleaved** to
improve compression. A sequence is grouped by bytes of length N, then the first
bytes of each group are written, then the second, and so on.

```
Original : ABCDabcd
N = 1    : ABCDabcd
N = 2    : ACacBDbd
N = 4    : AaBbCcDd
```

To reverse the process ("deinterleave"), the bytes can be interleaved by M,
where M is the the length of the bytes divided by N.

For a particular type, N is the byte length of a value of the type.

## Modes
[Modes]: #user-content-modes

The binary format has two "modes", which control certain conditions when
encoding or decoding.

**Place** mode serializes an entire game tree of a place. Indicated by the
`.rbxl` extension.

**Model** mode serializes a general tree of instances. Indicated by the `.rbxm`
extension.

# Format structure
The RBXL format consists of a signature, followed by a version number, then a
structure determined by the version.

Field     | Type        | Description
----------|-------------|------------
Signature | `[14]uint8` | A signature indicating the format.
Version   | `uint16`    | The version of the format.
Content   | `...`       | The structure determined by the version.

# Signature
An RBXL file begins with the signature, which is a constant sequence of the
following 14 bytes (in hexadecimal):

```
3C 72 6F 62 6C 6F 78 21 89 FF 0D 0A 1A 0A
```

This signature is similar to that of the [PNG][PNG] format.

Value                     | Description
--------------------------|------------
`3C 72 6F 62 6C 6F 78 21` | The sequence `<roblox!`, to signal the RBXL format.
`89 FF`                   | "Has the high bits set to detect transmission systems that do not support 8-bit data and to reduce the chance that a text file is mistakenly interpreted as a PNG, or vice versa."
`0D 0A`                   | "A DOS-style line ending (CRLF) to detect DOS-Unix line ending conversion of the data."
`1A`                      | "A byte that stops display of the file under DOS when the command type has been used—the end-of-file character."
`0A`                      | "A Unix-style line ending (LF) to detect Unix-DOS line ending conversion."

## Legacy format
The start of a file begins with `<roblox`. The next character is the **binary
marker**. When the binary marker is not the `!` character<VERIFY/>, the entire
file should instead be decoded as [RBXLX][RBXLX]. If RBXLX is not supported, an
error should be thrown instead.

Signature     | Action
--------------|-------
`<roblox!...` | Decode as RBXL.
`<roblox>...` | Decode as RBXLX.

RBXL files should be encoded only in the binary format. Implementations that
wish to use RBXLX format should do so explicitly, using the `.rbxlx` or `.rbxmx`
file extension to indicate the difference.

This document does not describe the RBXLX format, and as such, treats the binary
marker as a part of the signature.

## Version
After the signature, space is reserved for specifying a format version. While
there is currently only one version (version 0), this document is structured so
that multiple versions can be described, in case the format changes drastically.

For a particular version, the version number must match exactly.<VERIFY/> For
example, Version 0 expects the version number to be 0 (`00 00` as bytes). An
error should be thrown for unknown versions.

# Version 0
The RBXL format is similar to the [PNG][PNG] format, consisting of a header,
followed by a number of "chunks".

Field  | Type      | Description
-------|-----------|------------
Header | `Header`  | Contains data to aid with decoding.
Chunks | `[]Chunk` | A series of chunks.

The Chunks field is terminated by an [END][END] chunk. Encoders must end a file
with an END chunk, and decoders must return an error if the end of the file is
reached before an END chunk is decoded.

[PNG]: https://en.wikipedia.org/wiki/Portable_Network_Graphics#File_format

# Header
The **Header** structure contains the number of various elements within the
data.

Field         | Type       | Description
--------------|------------|------------
ClassCount    | `uint32`   | The number of unique classes encoded in the file.
InstanceCount | `uint32`   | The number of instances encoded in the file.
Reserved      | `[8]uint8` | Reserved for future use.

When decoding, the ClassCount and InstanceCount fields may be used to allocate
memory for performance purposes. Note that it is not guaranteed that these
values are respected by the remaining content of the file. Implementations that
choose to utilize this data should ignore or return an error for values that
exceed a reasonable threshold.

# Chunks
Following the header are a number of **Chunk** structures with the following
fields:

Field              | Type       | Description
-------------------|------------|------------
Signature          | `[4]uint8` | Determines the type of the chunk and structure of Payload.
CompressedLength   | `uint32`   | The length of Payload.
UncompressedLength | `uint32`   | The length of Payload after decompression.
Reserved           | `[4]uint8` | Reserved for future use.
Payload            | `[]uint8`  | The content of the chunk.

If CompressedLength is 0, then the Payload is uncompressed, and its length is
determined directly by UncompressedLength.

The payload of a chunk is compressed using [LZ4][LZ4].

[LZ4]: https://en.wikipedia.org/wiki/LZ4_(compression_algorithm)

## Chunk signatures
The following chunk types are defined:

Name                  | Signature | Bytes         | Description
----------------------|-----------|---------------|------------
[Metadata][META]      | `META`    | `4D 45 54 41` | Model metadata.
[SharedStrings][SSTR] | `SSTR`    | `53 53 54 52` | Shared string values.
[Instances][INST]     | `INST`    | `49 4E 53 54` | List of instances.
[Properties][PROP]    | `PROP`    | `50 52 4F 50` | Instance property data.
[Parent][PRNT]        | `PRNT`    | `50 52 4E 54` | Parent-child associations.
[End][END]            | `END.`    | `45 4E 44 00` | File terminator.

A decoder should return an error when it encounters an unknown chunk type. An
encoder should avoid encoding non-standard chunk types, as such signatures are
reserved for future expansion.<VERIFY/> An encoder may re-encode a decoded chunk
of an unknown type.

For improved compatibility, chunks should be expected in the following order:

1. Zero or one [META][META]
2. Zero or one [SSTR][SSTR]
3. Any number of [INST][INST]
4. Any number of [PROP][PROP]
5. One [PRNT][PRNT]
6. One [END][END]

## Metadata chunk
[META]: #user-content-metadata-chunk

A **Metadata** chunk payload contains an array that maps keys to values. It has
the following structure:

Field   | Type      | Description
--------|-----------|------------
Length  | `uint32`  | The length of the array.
Entries | `[]Entry` | A sequence of metadata entries, the length determined by the Length field.

An **Entry** structure has the following fields:

Field | Type     | Description
------|----------|------------
Key   | `string` | The metadata key.
Value | `string` | The metadata value.

### Metadata
The following metadata entries are known:

Key                  | Values | Description
---------------------|--------|------------
`ExplicitAutoJoints` | `true` | Model was made in a workspace with Explicit AutoJointsMode.

## SharedStrings chunk
[SSTR]: #user-content-sharedstrings-chunk

A **SharedStrings** chunk payload contains an array of strings that may be
shared by multiple properties. It has the following structure:

Field    | Type                  | Description
---------|-----------------------|------------
Reserved | `uint32`              | Reserved for future use.
Length   | `uint32`              | The number of strings in the chunk.
Strings  | `[]SharedStringValue` | The array of shared strings, the length determined by the Length field.

In [PROP][PROP] chunks with the [SharedString][SharedString] value type, values
are indices to the Strings array.

The **SharedStringValue** structure has the following fields:

Field    | Type        | Description
---------|-------------|------------
Hash     | `[16]uint8` | A digest of the value.
Value    | `string`    | The shared string.

Historically, the value of Hash was the MD5 digest of Value. Currently, it is
unused, with all bytes being 0.

[MD5]: https://en.wikipedia.org/wiki/Md5
[BLAKE2b]: https://en.wikipedia.org/wiki/BLAKE2b#BLAKE2b_algorithm

## Instances chunk
[INST]: #user-content-instances-chunk

An **Instances** chunk payload contains an array of instances of a single class.
It has the following structure:

Field       | Type      | Description
------------|-----------|------------
ClassID     | `int32`   | Identifies the class.
ClassName   | `string`  | The name of the class.
HasService  | `bool`    | Whether the chunk has service data.
Length      | `uint32`  | The number of instances in the chunk.
IDs         | `[]int32` | An array of IDs identifying each instance, the length determined by the Length field.
IsService   | `?[]bool` | An array of bools indicating whether the corresponding instance is a service, the length determined by the Length field.

The IsService field is present only if HasService is true. If all elements of
IsService would be false, then HasService should be false.

If encoding in [Model mode][Modes], then all instances should have their
IsService flag set to false.

<VERIFY q="If HasService is non-zero?"/>

## Properties chunk
[PROP]: #user-content-properties-chunk

A **Properties** chunk payload contains an array of values corresponding to one
property for each instance in an [INST][INST] chunk. It has the following
structure:

Field   | Type               | Description
--------|--------------------|------------
ClassID | `int32`            | Corresponds to the ClassID field of an INST chunk.
Name    | `string`           | The name of the property.
Values  | [`Values`][Values] | Contains the values of each property.

### Values
[Values]: #user-content-values

**Values** has the following structure:

Field   | Type                        | Description
--------|-----------------------------|------------
TypeID  | `uint8`                     | Determines the type of the value, and the structure of the Values field.
Values  | [`ValueArray`][value types] | A number of values.

The structure of **ValueArray** depends on the TypeID field. See [Value
types][Value types] for a description of each type.

Each type of ValueArray incorporates a length. This length is the Length field
of the [INST][INST] chunk corresponding the PROP chunk. This length will be
referred to as the "number of instances".

## Parents chunk
[PRNT]: #user-content-parents-chunk

A **Parents** chunk payload associates each instance with a parent instance. It
has the following structure:

Field    | Type                       | Description
---------|----------------------------|------------
Reserved | `uint8`                    | Reserved for future use.
Length   | `uint32`                   | The number of associations.
Children | [`References`][References] | An array of instance IDs indicating the child relation, the length determined by the Length field.
Parents  | [`References`][References] | An array of instance IDs indicating the parent relation, the length determined by the Length field.

The IDs in the Children and Parents arrays correspond to IDs in previously
decoded [INST][INST] chunks.

When decoding, each instance in the Children array has its Parent property set
to the corresponding instance in the Parents array.

## End chunk
[END]: #user-content-end-chunk

An **End** chunk signals the end of the file, terminating the parsing of chunks.
The payload is an unstructured sequence of bytes:

Field    | Type      | Description
---------|-----------|------------
Payload  | `[]uint8` | A sequence of bytes.

For improved compatibility, an END chunk should be encoded uncompressed, and the
payload should contain the exact string `</roblox>`, or the following sequence
of bytes in hexadecimal:

```
3C 2F 72 6F 62 6C 6F 78 3E
```

It is not necessary to consider the content of the payload when decoding. The
payload is used for compatibility when transferring and storing the file.

# Value types
[Value types]: #user-content-value-types

This section describes the structure of property value types within [PROP][PROP]
chunks.

The following table provides an overview:

| ID         | Name                                     |
|------------|------------------------------------------|
| 0x00       | (invalid)                                |
| 0x01       | [String][String]                         |
| 0x02       | [Bool][Bool]                             |
| 0x03       | [Int][Int]                               |
| 0x04       | [Float][Float]                           |
| 0x05       | [Double][Double]                         |
| 0x06       | [UDim][UDim]                             |
| 0x07       | [UDim2][UDim2]                           |
| 0x08       | [Ray][Ray]                               |
| 0x09       | [Faces][Faces]                           |
| 0x0A       | [Axes][Axes]                             |
| 0x0B       | [BrickColor][BrickColor]                 |
| 0x0C       | [Color3][Color3]                         |
| 0x0D       | [Vector2][Vector2]                       |
| 0x0E       | [Vector3][Vector3]                       |
| 0x0F       | [Vector2int16][Vector2int16]             |
| 0x10       | [CFrame][CFrame]                         |
| 0x11       | [CFrameQuat][CFrameQuat]                 |
| 0x12       | [Token][Token]                           |
| 0x13       | [Reference][Reference]                   |
| 0x14       | [Vector3int16][Vector3int16]             |
| 0x15       | [NumberSequence][NumberSequence]         |
| 0x16       | [ColorSequence][ColorSequence]           |
| 0x17       | [NumberRange][NumberRange]               |
| 0x18       | [Rect][Rect]                             |
| 0x19       | [PhysicalProperties][PhysicalProperties] |
| 0x1A       | [Color3uint8][Color3uint8]               |
| 0x1B       | [Int64][Int64]                           |
| 0x1C       | [SharedString][SharedString]             |
| 0x1D       | (reserved)                               |
| 0x1E       | [Optional][Optional]                     |
| 0x1F       | [UniqueId][UniqueId]                     |
| 0x20..0xFF | (reserved)                               |

The ID is the value of a [Values.TypeID][Values] field. The remaining IDs are
reserved for future use. A decoder should return an error if it encounters an
invalid or reserved type ID.

Each following subsection describes a **Type**, indicating the type of
`ValueArray` for the type ID.

## String
[String]: #user-content-string

Corresponds to string-like types.

- **ID**: 0x01
- **Type**: [`[]string`][string]

Elements correspond to one of the following Roblox data types:

- string
- BinaryString
- ProtectedString
- Content

To properly decode back into the original string type, external information not
available in the format will be required (e.g. class descriptors).

## Bool
[Bool]: #user-content-bool

Corresponds to the "bool" Roblox data type.

- **ID**: 0x02
- **Type**: `[]bool`

## Int
[Int]: #user-content-int

Corresponds to the "int" Roblox data type.

- **ID**: 0x03
- **Type**: `[]zint32b~4`

## Float
[Float]: #user-content-float

Corresponds to the "float" Roblox data type.

- **ID**: 0x04
- **Type**: `[]rfloat32b~4`

## Double
[Double]: #user-content-double

Corresponds to the "double" Roblox data type.

- **ID**: 0x05
- **Type**: `[]float64`

## UDim
[UDim]: #user-content-udim

Corresponds to the "UDim" Roblox data type.

- **ID**: 0x06
- **Type**: `[]UDim~8`

A **UDim** is a structure with the following fields:

Field  | Type       | Description
-------|------------|------------
Scale  | `rfloat32` | Corresponds to `UDim.Scale`.
Offset | `zint32`   | Corresponds to `UDim.Offset`.

## UDim2
[UDim2]: #user-content-udim2

Corresponds to the "UDim2" Roblox data type.

- **ID**: 0x07
- **Type**: `[]UDim2~16`

A **UDim2** is a structure with the following fields:

Field   | Type       | Description
--------|------------|------------
ScaleX  | `rfloat32` | Corresponds to `UDim2.X.Scale`.
ScaleY  | `rfloat32` | Corresponds to `UDim2.Y.Scale`.
OffsetX | `zint32`   | Corresponds to `UDim2.X.Offset`.
OffsetY | `zint32`   | Corresponds to `UDim2.Y.Offset`.

## Ray
[Ray]: #user-content-ray

Corresponds to the "Ray" Roblox data type.

- **ID**: 0x08
- **Type**: `[]Ray`

A **Ray** is a structure with the following fields:

Field      | Type      | Description
-----------|-----------|------------
OriginX    | `float32` | Corresponds to `Ray.Origin.X`.
OriginY    | `float32` | Corresponds to `Ray.Origin.Y`.
OriginZ    | `float32` | Corresponds to `Ray.Origin.Z`.
DirectionX | `float32` | Corresponds to `Ray.Direction.X`.
DirectionY | `float32` | Corresponds to `Ray.Direction.Y`.
DirectionZ | `float32` | Corresponds to `Ray.Direction.Z`.

## Faces
[Faces]: #user-content-faces

Corresponds to the "Faces" Roblox data type.

- **ID**: 0x09
- **Type**: `[]uint8`

Each component occupies one bit:

Field  | Bit | Description
-------|----:|------------
Right  |   0 | Corresponds to `Faces.Right`.
Top    |   1 | Corresponds to `Faces.Top`.
Back   |   2 | Corresponds to `Faces.Back`.
Left   |   3 | Corresponds to `Faces.Left`.
Bottom |   4 | Corresponds to `Faces.Bottom`.
Front  |   5 | Corresponds to `Faces.Front`.
_      |   6 | Unused.
_      |   7 | Unused.

## Axes
[Axes]: #user-content-axes

Corresponds to the "Axes" Roblox data type.

- **ID**: 0x0A
- **Type**: `[]uint8`

Each component occupies one bit:

Field | Bit | Description
------|----:|------------
X     |   0 | Corresponds to `Axes.X`.
Y     |   1 | Corresponds to `Axes.Y`.
Z     |   2 | Corresponds to `Axes.Z`.
_     |   3 | Unused.
_     |   4 | Unused.
_     |   5 | Unused.
_     |   6 | Unused.
_     |   7 | Unused.

## BrickColor
[BrickColor]: #user-content-brickcolor

Corresponds to the "BrickColor" Roblox data type.

- **ID**: 0x0B
- **Type**: `[]uint32b~4`

Each element corresponds to the Number of a BrickColor.

## Color3
[Color3]: #user-content-color3

Corresponds to the "Color3" Roblox data type.

- **ID**: 0x0C
- **Type**: `[]Color3~12`

A **Color3** is a structure with the following fields:

Field | Type       | Description
------|------------|------------
R     | `rfloat32` | Corresponds to `Color3.R`.
G     | `rfloat32` | Corresponds to `Color3.G`.
B     | `rfloat32` | Corresponds to `Color3.B`.

## Vector2
[Vector2]: #user-content-vector2

Corresponds to the "Vector2" Roblox data type.

- **ID**: 0x0D
- **Type**: `[]Vector2~8`

A **Vector2** is a structure with the following fields:

Field | Type       | Description
------|------------|------------
X     | `rfloat32` | Corresponds to `Vector2.X`.
Y     | `rfloat32` | Corresponds to `Vector2.Y`.

## Vector3
[Vector3]: #user-content-vector3

Corresponds to the "Vector3" Roblox data type.

- **ID**: 0x0E
- **Type**: `[]Vector3~12`

A **Vector3** is a structure with the following fields:

Field | Type       | Description
------|------------|------------
X     | `rfloat32` | Corresponds to `Vector3.X`.
Y     | `rfloat32` | Corresponds to `Vector3.Y`.
Z     | `rfloat32` | Corresponds to `Vector3.Z`.

## Vector2int16
[Vector2int16]: #user-content-vector2int16

Corresponds to the "Vector2int16" Roblox data type.

- **ID**: 0x0F
- **Type**: `[]Vector2int16`

A **Vector2int16** is a structure with the following fields:

Field | Type    | Description
------|---------|------------
X     | `int16` | Corresponds to `Vector2int16.X`.
Y     | `int16` | Corresponds to `Vector2int16.Y`.

## CFrame
[CFrame]: #user-content-cframe

Corresponds to the "CFrame" Roblox data type.

- **ID**: 0x10
- **Type**: `CFrames`

A **CFrames** is a structure with the following fields:

Field    | Type            | Description
---------|-----------------|------------
Rotation | `[]Matrix`      | The rotation parts of each CFrame.
Position | `[]Vector3~12` | The position parts of each CFrame.

The length of each array is equal to the number of instances.

A **Matrix** is a structure with the following fields:

Field    | Type          | Description
---------|---------------|------------
ID       | `uint8`       | A value representing a predefined rotation matrix.
Values   | `?[9]float32` | Rotation matrix data. Present only if ID is 0.

The indices of the array correspond to the following matrix elements:

| Right | Up | -Look |
|------:|---:|------:|
|     0 |  1 |     2 |
|     3 |  4 |     5 |
|     6 |  7 |     8 |

Or, expressed as a CFrame constructor:

```lua
CFrame.new(_, _, _, 0, 1, 2, 3, 4, 5, 6, 7, 8)
```

## Rotation IDs
[RotationIDs]: #user-content-rotation-ids

The following IDs must produce the corresponding rotation matrix. Non-zero IDs
that aren't in this list are undefined.

```
0x02 : [+1 +0 +0 +0 +1 +0 +0 +0 +1]
0x03 : [+1 +0 +0 +0 +0 -1 +0 +1 +0]
0x05 : [+1 +0 +0 +0 -1 +0 +0 +0 -1]
0x06 : [+1 +0 -0 +0 +0 +1 +0 -1 +0]
0x07 : [+0 +1 +0 +1 +0 +0 +0 +0 -1]
0x09 : [+0 +0 +1 +1 +0 +0 +0 +1 +0]
0x0A : [+0 -1 +0 +1 +0 -0 +0 +0 +1]
0x0C : [+0 +0 -1 +1 +0 +0 +0 -1 +0]
0x0D : [+0 +1 +0 +0 +0 +1 +1 +0 +0]
0x0E : [+0 +0 -1 +0 +1 +0 +1 +0 +0]
0x10 : [+0 -1 +0 +0 +0 -1 +1 +0 +0]
0x11 : [+0 +0 +1 +0 -1 +0 +1 +0 -0]
0x14 : [-1 +0 +0 +0 +1 +0 +0 +0 -1]
0x15 : [-1 +0 +0 +0 +0 +1 +0 +1 -0]
0x17 : [-1 +0 +0 +0 -1 +0 +0 +0 +1]
0x18 : [-1 +0 -0 +0 +0 -1 +0 -1 -0]
0x19 : [+0 +1 -0 -1 +0 +0 +0 +0 +1]
0x1B : [+0 +0 -1 -1 +0 +0 +0 +1 +0]
0x1C : [+0 -1 -0 -1 +0 -0 +0 +0 -1]
0x1E : [+0 +0 +1 -1 +0 +0 +0 -1 +0]
0x1F : [+0 +1 +0 +0 +0 -1 -1 +0 +0]
0x20 : [+0 +0 +1 +0 +1 -0 -1 +0 +0]
0x22 : [+0 -1 +0 +0 +0 +1 -1 +0 +0]
0x23 : [+0 +0 -1 +0 -1 -0 -1 +0 -0]
```

## CFrameQuat
[CFrameQuat]: #user-content-cframequat

Corresponds to a quaternion representation of the "CFrame" Roblox data type.

- **ID**: 0x11
- **Type**: `CFrameQuats`

*In practice, this type is not used.*

A **CFrameQuats** is a structure with the following fields:

Field    | Type            | Description
---------|-----------------|------------
Rotation | `[]Quat`        | The rotation parts of each CFrame.
Position | `[]Vector3~12` | The position parts of each CFrame.

The length of each array is equal to the number of instances.

A **Quat** is a structure with the following fields:

Field    | Type       | Description
---------|------------|------------
ID       | `uint8`    | A value representing a predefined rotation matrix.
QX       | `?float32` | The X component of the quaternion. Present only if ID is 0.
QY       | `?float32` | The Y component of the quaternion. Present only if ID is 0.
QZ       | `?float32` | The Z component of the quaternion. Present only if ID is 0.
QW       | `?float32` | The W component of the quaternion. Present only if ID is 0.

The components of the quaternion are expected to be converted to a rotation
matrix. The ID is the same as in the [CFrame][CFrame] type.

## Token
[Token]: #user-content-token

Corresponds to the "EnumItem" Roblox data type.

- **ID**: 0x12
- **Type**: `[]uint32b~4`

Each element is the value of an enum item. The associated enum is not specified
in the format, and must be determined from an external source (e.g. class
descriptors).

## Reference
[Reference]: #user-content-reference

Corresponds to the "Instance" Roblox data type.

- **ID**: 0x13
- **Type**: [`References`][References]

Each element is the ID of an instance within the file.

## Vector3int16
[Vector3int16]: #user-content-vector3int16

Corresponds to the "Vector3int16" Roblox data type.

- **ID**: 0x14
- **Type**: `[]Vector3int16`

A **Vector3int16** is a structure with the following fields:

Field | Type    | Description
------|---------|------------
X     | `int16` | Corresponds to `Vector3int16.X`.
Y     | `int16` | Corresponds to `Vector3int16.Y`.
Z     | `int16` | Corresponds to `Vector3int16.Z`.

## NumberSequence
[NumberSequence]: #user-content-numbersequence

Corresponds to the "NumberSequence" Roblox data type.

- **ID**: 0x15
- **Type**: `[]NumberSequence`

A **NumberSequence** is a structure with the following fields:

Field     | Type                       | Description
----------|----------------------------|------------
Length    | `uint32`                   | The length of the sequence.
Keypoints | `[]NumberSequenceKeypoint` | The keypoints of the sequence, the length determined by the Length field.

A **NumberSequenceKeypoint** is a structure with the following fields:

Field    | Type      | Description
---------|-----------|------------
Time     | `float32` | Corresponds to `NumberSequenceKeypoint.Time`.
Value    | `float32` | Corresponds to `NumberSequenceKeypoint.Value`.
Envelope | `float32` | Corresponds to `NumberSequenceKeypoint.Envelope`.

## ColorSequence
[ColorSequence]: #user-content-colorsequence

Corresponds to the "ColorSequence" Roblox data type.

- **ID**: 0x16
- **Type**: `[]ColorSequence`

A **ColorSequence** is a structure with the following fields:

Field     | Type                      | Description
----------|---------------------------|------------
Length    | `uint32`                  | The length of the sequence.
Keypoints | `[]ColorSequenceKeypoint` | The keypoints of the sequence, the length determined by the Length field.

A **ColorSequenceKeypoint** is a structure with the following fields:

Field    | Type               | Description
---------|--------------------|------------
Time     | `float32`          | Corresponds to `ColorSequenceKeypoint.Time`.
Value    | [`Color3`][Color3] | Corresponds to `ColorSequenceKeypoint.Value`.
Envelope | `float32`          | Corresponds to `ColorSequenceKeypoint.Envelope`.

## NumberRange
[NumberRange]: #user-content-numberrange

Corresponds to the "NumberRange" Roblox data type.

- **ID**: 0x17
- **Type**: `[]NumberRange`

A **NumberRange** is a structure with the following fields:

Field | Type      | Description
------|-----------|------------
Min   | `float32` | Corresponds to `NumberRange.Min`.
Max   | `float32` | Corresponds to `NumberRange.Max`.

## Rect
[Rect]: #user-content-rect

Corresponds to the "Rect" Roblox data type.

- **ID**: 0x18
- **Type**: `[]Rect~16`

A **Rect** is a structure with the following fields:

Field | Type                 | Description
------|----------------------|------------
Min   | [`Vector2`][Vector2] | Corresponds to `Rect.Min`.
Max   | [`Vector2`][Vector2] | Corresponds to `Rect.Max`.

## PhysicalProperties
[PhysicalProperties]: #user-content-physicalproperties

Corresponds to the "PhysicalProperties" Roblox data type.

- **ID**: 0x19
- **Type**: `[]PhysicalProperties`

A **PhysicalProperties** is a structure with the following fields:

Field            | Type       | Description
-----------------|------------|------------
CustomPhysics    | `bool`     | Whether the value has custom physics.
Density          | `?float32` | Present if CustomPhysics is true. Corresponds to `PhysicalProperties.Density`.
Friction         | `?float32` | Present if CustomPhysics is true. Corresponds to `PhysicalProperties.Friction`.
Elasticity       | `?float32` | Present if CustomPhysics is true. Corresponds to `PhysicalProperties.Elasticity`.
FrictionWeight   | `?float32` | Present if CustomPhysics is true. Corresponds to `PhysicalProperties.FrictionWeight`.
ElasticityWeight | `?float32` | Present if CustomPhysics is true. Corresponds to `PhysicalProperties.ElasticityWeight`.

## Color3uint8
[Color3uint8]: #user-content-color3uint8

Corresponds to the "Color3uint8" Roblox data type.

- **ID**: 0x1A
- **Type**: `[]Color3uint8~3`

A **Color3uint8** is a structure with the following fields:

Field | Type    | Description
------|---------|------------
R     | `uint8` | Corresponds to `Color3uint8.R`.
G     | `uint8` | Corresponds to `Color3uint8.G`.
B     | `uint8` | Corresponds to `Color3uint8.B`.


## Int64
[Int64]: #user-content-int64

Corresponds to the "int64" Roblox data type.

- **ID**: 0x1B
- **Type**: `[]zint64b~8`

## SharedString
[SharedString]: #user-content-sharedstring

Corresponds to the "SharedString" Roblox data type.

- **ID**: 0x1C
- **Type**: `[]uint32b~4`

Each element is an index of the [SharedStrings.Strings][SSTR] array.

## Optional
[Optional]: #user-content-optional

Corresponds to an optional Roblox data type.

- **ID**: 0x1E
- **Type**: `Optional`

An **Optional** is a structure with the following fields:

Field    | Type               | Description
---------|--------------------|------------
Values   | [`Values`][Values] | Any valid type.
Presence | [`Values`][Values] | Always of the [Bool][Bool] type.

The length of each array is equal to the number of instances. Each value in
Presence indicates whether the corresponding value in Values is present.

When encoding a value that isn't present, the zero for the type is used.

Currently, only optional CFrames are known to be in use. Other types may throw
an error.

## UniqueId
[UniqueId]: #user-content-uniqueid

Corresponds the the "UniqueId" Roblox data type.

- **ID**: 0x1F
- **Type**: `[]UniqueId~16`

A **UniqueId** is a structure with the following fields:

Field    | Type     | Description
---------|----------|------------
Index    | `uint32` | The sequential portion of the ID.
Time     | `uint32` | The time portion of the ID.
Random   | `zint64` | The random portion of the ID.

If encoding while not in [Place mode][Modes], properties of this type should be
skipped.
