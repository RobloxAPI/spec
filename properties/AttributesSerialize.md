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
`[]T`    | `[]uint8`    | An array with elements of type `T`, the length of which is determined dynamically.
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
[Dictionary]: #user-content-dictionary

The **Dictionary** type represents a collection of keys mapped to values. It is
a structure with the following fields:

Field   | Type               | Description
--------|--------------------|------------
Length  | `uint32`           | The number of entries in the dictionary.
Entries | [`[]Entry`][Entry] | The entries of the dictionary. The length is determined by the Length field.

## Entry
[Entry]: #user-content-entry

The **Entry** type represents a single key-value pair from a
[Dictionary][Dictionary]. It is a structure with the following fields:

Field | Type               | Description
------|--------------------|------------
Key   | [`String`][String] | The key of the entry.
Value | [`Value`][Value]   | The value of the entry.

## Value
[Value]: #user-content-value

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
|       23 | [NumberSequence][NumberSequence] |
|       25 | [ColorSequence][ColorSequence]   |
|       27 | [NumberRange][NumberRange]       |
|       28 | [Rect][Rect]                     |

Other IDs are reserved for future use.

## Value types

### String
[String]: #user-content-string

The **String** type corresponds to the "string" Roblox data type.

It is a structure with the following fields:

Field   | Type      | Description
--------|-----------|------------
Length  | `uint32`  | The length of the string, in bytes.
Content | `[]uint8` | The content of the string. The length is determined by the Length field.

### Bool
[Bool]: #user-content-bool

The **Bool** type corresponds to the "bool" Roblox data type.

**Type:** `uint8`

A value of 0 represents false, while 1 represents true. Any other value may be
interpreted as true.

### Float
[Float]: #user-content-float

The **Float** type corresponds to the "float" Roblox data type.

**Type:** `float32`

While Studio is not capable of producing attributes of the Float type
([Double][Double] is used for numeric types), it does accept the type when
decoding.

### Double
[Double]: #user-content-double

The **Double** type corresponds to the "double" Roblox data type.

**Type:** `float64`

### UDim
[UDim]: #user-content-udim

The **UDim** type corresponds to the "UDim" Roblox data type. It is a structure
with the following fields:

Field  | Type      | Description
-------|-----------|------------
Scale  | `float32` | Corresponds to `UDim.Scale`.
Offset | `int32`   | Corresponds to `UDim.Offset`.


### UDim2
[UDim2]: #user-content-udim2

The **UDim2** type corresponds to the "UDim2" Roblox data type. It is a
structure with the following fields:

Field | Type           | Description
------|----------------|------------
X     | [`UDim`][UDim] | Corresponds to `UDim2.X`.
Y     | [`UDim`][UDim] | Corresponds to `UDim2.Y`.

### BrickColor
[BrickColor]: #user-content-brickcolor

The **BrickColor** type corresponds to the "BrickColor" Roblox data type.

**Type:** `uint32`

### Color3
[Color3]: #user-content-color3

The **Color3** type corresponds to the "Color3" Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
R     | `float32` | Corresponds to `Color3.R`.
G     | `float32` | Corresponds to `Color3.G`.
B     | `float32` | Corresponds to `Color3.B`.

### Vector2
[Vector2]: #user-content-vector2

The **Vector2** type corresponds to the "Vector2" Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
X     | `float32` | Corresponds to `Vector2.X`.
Y     | `float32` | Corresponds to `Vector2.Y`.

### Vector3
[Vector3]: #user-content-vector3

The **Vector3** type corresponds to the "Vector3" Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
X     | `float32` | Corresponds to `Vector3.X`.
Y     | `float32` | Corresponds to `Vector3.Y`.
Z     | `float32` | Corresponds to `Vector3.Z`.

### NumberSequence
[NumberSequence]: #user-content-numbersequence

The **NumberSequence** type corresponds to the "NumberSequence" Roblox data
type. It is a structure with the following fields:

Field     | Type                                                 | Description
----------|------------------------------------------------------|------------
Length    | `uint32`                                             | The length of the sequence.
Keypoints | [`[]NumberSequenceKeypoint`][NumberSequenceKeypoint] | The keypoints of the sequence, the length determined by the Length field.

### NumberSequenceKeypoint
[NumberSequenceKeypoint]: #user-content-numbersequencekeypoint

The **NumberSequenceKeypoint** type corresponds to the "NumberSequenceKeypoint"
Roblox data type. It is a structure with the following fields:

Field    | Type      | Description
---------|-----------|------------
Envelope | `float32` | Corresponds to `NumberSequenceKeypoint.Envelope`.
Time     | `float32` | Corresponds to `NumberSequenceKeypoint.Time`.
Value    | `float32` | Corresponds to `NumberSequenceKeypoint.Value`.

### ColorSequence
[ColorSequence]: #user-content-colorsequence

The **ColorSequence** type corresponds to the "ColorSequence" Roblox data type.
It is a structure with the following fields:

Field     | Type                                               | Description
----------|----------------------------------------------------|------------
Length    | `uint32`                                           | The length of the sequence.
Keypoints | [`[]ColorSequenceKeypoint`][ColorSequenceKeypoint] | The keypoints of the sequence, the length determined by the Length field.

### ColorSequenceKeypoint
[ColorSequenceKeypoint]: #user-content-colorsequencekeypoint

The **ColorSequenceKeypoint** type corresponds to the "ColorSequenceKeypoint"
Roblox data type. It is a structure with the following fields:

Field    | Type               | Description
---------|--------------------|------------
Envelope | `float32`          | Corresponds to `ColorSequenceKeypoint.Envelope`.
Time     | `float32`          | Corresponds to `ColorSequenceKeypoint.Time`.
Value    | [`Color3`][Color3] | Corresponds to `ColorSequenceKeypoint.Value`.

### NumberRange
[NumberRange]: #user-content-numberrange

The **NumberRange** type corresponds to the "NumberRange" Roblox data type. It
is a structure with the following fields:

Field | Type      | Description
------|-----------|------------
Min   | `float32` | Corresponds to `NumberRange.Min`.
Max   | `float32` | Corresponds to `NumberRange.Max`.

### Rect
[Rect]: #user-content-rect

The **Rect** type corresponds to the "Rect" Roblox data type. It is a structure
with the following fields:

Field | Type                 | Description
------|----------------------|------------
Min   | [`Vector2`][Vector2] | Corresponds to `Rect.Min`.
Max   | [`Vector2`][Vector2] | Corresponds to `Rect.Max`.
