# Instance AttributesSerialize format
This document describes the binary format of the `Instance.AttributesSerialize`
property.

# Type notation
The following primitive types are defined for use throughout this document:

Type     | Example      | Description
---------|--------------|------------
`intN`   | `int32`      | A signed integer with a size of `N` bits. Little-endian encoding.
`uintN`  | `uint8`      | An unsigned signed integer with a size of `N` bits. Little-endian encoding.
`floatN` | `float32`    | A IEEE 754 floating-point number with a size of `N` bits. Little-endian encoding.
`[N]T`   | `[9]float32` | An array of constant length `N`, with elements of type `T`.
`[]T`    | `[]uint8`    | An array with elements of type `T`, the length of which is determined dynamically.
`?T`     | `?uint32`    | A value of type `T`, which may or may not be present based on some condition. If not present, the value is omitted entirely.
`...`    | `...`        | The type is determined dynamically.

A type may also be a named type defined elsewhere in the document.

## Structures
A "structure" type comprises a number of ordered fields, each having a certain
type. This document describes structures by using a table with Field, Type and
Description columns. The Field column contains a name that identifies the field.
The Type column contains a type as described above. The Description column
contains a description of the field. Each row of the table is a specific field.
When decoding or encoding, each row is read or written in order, top to bottom.

# Format structure
The attributes format consists of one [Dictionary][Dictionary] value, which maps
attribute names to values.

Several limitations may be applied to the Key field of an [Entry][Entry] in this
Dictionary:

- Must not have a length greater than 100 bytes.
- May only contain alphanumeric bytes and underscores. (`^[0-9A-Za-z_]*$`)
- Must not begin with `RBX`, which is reserved for Roblox.

## Dictionary
[Dictionary]: #dictionary

The **Dictionary** type represents a collection of keys mapped to values. It is
a structure with the following fields:

Field   | Type               | Description
--------|--------------------|------------
Length  | `uint32`           | The number of entries in the dictionary.
Entries | [`[]Entry`][Entry] | The entries of the dictionary. The length is determined by the Length field.

## Entry
[Entry]: #entry

The **Entry** type represents a single key-value pair from a
[Dictionary][Dictionary]. It is a structure with the following fields:

Field | Type               | Description
------|--------------------|------------
Key   | [`String`][String] | The key of the entry.
Value | [`Value`][Value]   | The value of the entry.

## Value
[Value]: #value

The **Value** type holds a value of one of a number of types. It is a structure
with the following fields:

Field  | Type     | Description
-------|----------|------------
Type   | `uint8`  | An identifier indicating the value's type.
Value  | `...`    | Type determined by the Type field.

The following table describes the identifier's corresponding type:

|  Type ID | Type                             |
|---------:|----------------------------------|
|        2 | [String][String]                 |
|        3 | [Bool][Bool]                     |
|        5 | [Float][Float]                   |
|        6 | [Double][Double]                 |
|        9 | [UDim][UDim]                     |
|       10 | [UDim2][UDim2]                   |
|       14 | [BrickColor][BrickColor]         |
|       15 | [Color3][Color3]                 |
|       16 | [Vector2][Vector2]               |
|       17 | [Vector3][Vector3]               |
|       20 | [CFrame][CFrame]                 |
|       23 | [NumberSequence][NumberSequence] |
|       25 | [ColorSequence][ColorSequence]   |
|       27 | [NumberRange][NumberRange]       |
|       28 | [Rect][Rect]                     |

Other IDs are reserved for future use.

## Value types

### String
[String]: #string

The **String** type corresponds to the "string" Roblox data type.

It is a structure with the following fields:

Field   | Type      | Description
--------|-----------|------------
Length  | `uint32`  | The length of the string, in bytes.
Content | `[]uint8` | The content of the string. The length is determined by the Length field.

### Bool
[Bool]: #bool

The **Bool** type corresponds to the "bool" Roblox data type.

**Type:** `uint8`

A value of 0 represents false, while 1 represents true. Any other value may be
interpreted as true.

### Float
[Float]: #float

The **Float** type corresponds to the "float" Roblox data type.

**Type:** `float32`

While Studio is not capable of producing attributes of the Float type
([Double][Double] is used for numeric types), it does accept the type when
decoding.

### Double
[Double]: #double

The **Double** type corresponds to the "double" Roblox data type.

**Type:** `float64`

### UDim
[UDim]: #udim

The **UDim** type corresponds to the "UDim" Roblox data type. It is a structure
with the following fields:

Field  | Type      | Description
-------|-----------|------------
Scale  | `float32` | Corresponds to `UDim.Scale`.
Offset | `int32`   | Corresponds to `UDim.Offset`.


### UDim2
[UDim2]: #udim2

The **UDim2** type corresponds to the "UDim2" Roblox data type. It is a
structure with the following fields:

Field | Type           | Description
------|----------------|------------
X     | [`UDim`][UDim] | Corresponds to `UDim2.X`.
Y     | [`UDim`][UDim] | Corresponds to `UDim2.Y`.

### BrickColor
[BrickColor]: #brickcolor

The **BrickColor** type corresponds to the "BrickColor" Roblox data type.

**Type:** `uint32`

### Color3
[Color3]: #color3

The **Color3** type corresponds to the "Color3" Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
R     | `float32` | Corresponds to `Color3.R`.
G     | `float32` | Corresponds to `Color3.G`.
B     | `float32` | Corresponds to `Color3.B`.

### Vector2
[Vector2]: #vector2

The **Vector2** type corresponds to the "Vector2" Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
X     | `float32` | Corresponds to `Vector2.X`.
Y     | `float32` | Corresponds to `Vector2.Y`.

### Vector3
[Vector3]: #vector3

The **Vector3** type corresponds to the "Vector3" Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
X     | `float32` | Corresponds to `Vector3.X`.
Y     | `float32` | Corresponds to `Vector3.Y`.
Z     | `float32` | Corresponds to `Vector3.Z`.

### CFrame
[CFrame]: #cframe

The **CFrame** type corresponds to the `CFrame` Roblox data type. It is a
structure with the following fields:

Field    | Type                 | Description
---------|----------------------|------------
Position | [`Vector3`][Vector3] | The position portion of the CFrame.
ID       | `uint8`              | A value representing a prefined rotation matrix.
Rotation | `?[9]float32`        | The rotation portion of the CFrame. Present only if ID is 0.

The indices of the rotation array correspond to the following matrix elements:

| Right | Up | -Look |
|------:|---:|------:|
|     0 |  1 |     2 |
|     3 |  4 |     5 |
|     6 |  7 |     8 |

Or, expressed as a CFrame constructor:

```lua
CFrame.new(_, _, _, 0, 1, 2, 3, 4, 5, 6, 7, 8)
```

#### Rotation IDs
[RotationIDs]: #rotation-ids

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

### NumberSequence
[NumberSequence]: #numbersequence

The **NumberSequence** type corresponds to the "NumberSequence" Roblox data
type. It is a structure with the following fields:

Field     | Type                                                 | Description
----------|------------------------------------------------------|------------
Length    | `uint32`                                             | The length of the sequence.
Keypoints | [`[]NumberSequenceKeypoint`][NumberSequenceKeypoint] | The keypoints of the sequence, the length determined by the Length field.

### NumberSequenceKeypoint
[NumberSequenceKeypoint]: #numbersequencekeypoint

The **NumberSequenceKeypoint** type corresponds to the "NumberSequenceKeypoint"
Roblox data type. It is a structure with the following fields:

Field    | Type      | Description
---------|-----------|------------
Envelope | `float32` | Corresponds to `NumberSequenceKeypoint.Envelope`.
Time     | `float32` | Corresponds to `NumberSequenceKeypoint.Time`.
Value    | `float32` | Corresponds to `NumberSequenceKeypoint.Value`.

### ColorSequence
[ColorSequence]: #colorsequence

The **ColorSequence** type corresponds to the "ColorSequence" Roblox data type.
It is a structure with the following fields:

Field     | Type                                               | Description
----------|----------------------------------------------------|------------
Length    | `uint32`                                           | The length of the sequence.
Keypoints | [`[]ColorSequenceKeypoint`][ColorSequenceKeypoint] | The keypoints of the sequence, the length determined by the Length field.

### ColorSequenceKeypoint
[ColorSequenceKeypoint]: #colorsequencekeypoint

The **ColorSequenceKeypoint** type corresponds to the "ColorSequenceKeypoint"
Roblox data type. It is a structure with the following fields:

Field    | Type               | Description
---------|--------------------|------------
Envelope | `float32`          | Corresponds to `ColorSequenceKeypoint.Envelope`.
Time     | `float32`          | Corresponds to `ColorSequenceKeypoint.Time`.
Value    | [`Color3`][Color3] | Corresponds to `ColorSequenceKeypoint.Value`.

### NumberRange
[NumberRange]: #numberrange

The **NumberRange** type corresponds to the "NumberRange" Roblox data type. It
is a structure with the following fields:

Field | Type      | Description
------|-----------|------------
Min   | `float32` | Corresponds to `NumberRange.Min`.
Max   | `float32` | Corresponds to `NumberRange.Max`.

### Rect
[Rect]: #rect

The **Rect** type corresponds to the "Rect" Roblox data type. It is a structure
with the following fields:

Field | Type                 | Description
------|----------------------|------------
Min   | [`Vector2`][Vector2] | Corresponds to `Rect.Min`.
Max   | [`Vector2`][Vector2] | Corresponds to `Rect.Max`.
