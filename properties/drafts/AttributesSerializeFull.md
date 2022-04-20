*DRAFT: Contains additional types that were discovered, but are not officially
supported.*

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
`[N]T`   | `[9]float32` | An array with a constant length of `N`, with elements of type `T`.
`[]T`    | `[]uint8`    | An array with elements of type `T`, the length of which is determined dynamically.
`?T`     | `?uint8`     | A value of type `T`, which may or may not be present based on some condition. If not present, the value is omitted entirely.
`...`    | `...`        | The type is determined dynamically.

A type may also be a named type defined elsewhere in the document.

## Structures
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

# Format structure
The attributes format consists of one [Dictionary][Dictionary] value, which maps
attribute names to values.

Several limitations may be applied to the Key field of an [Entry][Entry] in this
Dictionary:

- Must not have a length greater than 100 bytes.
- May only contain alphanumeric bytes and underscores. (`^[0-9A-Za-z_]*$`)
- Must not begin with `RBX`, which is reserved for Roblox.

## Value
[Value]: #user-content-value

The **Value** type holds a value of one of a number of types. It is a structure
with the following fields:

Field  | Type     | Description
-------|----------|------------
Type   | `uint8`  | An identifier indicating the value's type.
Value  | `...`    | Type determined by the Type field.

The following table describes the identifier's corresponding type:

|  Type ID | Type                                             | Supported
|---------:|--------------------------------------------------|----------
|        0 | (reserved)                                       |
|        1 | (reserved)                                       |
|        2 | [String][String]                                 | ✔
|        3 | [Bool][Bool]                                     | ✔
|        4 | [Int][Int]                                       | ❌
|        5 | [Float][Float]                                   | ❌
|        6 | [Double][Double]                                 | ✔
|        7 | [Array][Array]                                   | ❌
|        8 | [Dictionary][Dictionary]                         | ❌
|        9 | [UDim][UDim]                                     | ✔
|       10 | [UDim2][UDim2]                                   | ✔
|       11 | [Ray][Ray]                                       | ❌
|       12 | [Faces][Faces]                                   | ❌
|       13 | [Axes][Axes]                                     | ❌
|       14 | [BrickColor][BrickColor]                         | ✔
|       15 | [Color3][Color3]                                 | ✔
|       16 | [Vector2][Vector2]                               | ✔
|       17 | [Vector3][Vector3]                               | ✔
|       18 | [Vector2int16][Vector2int16]                     | ❌
|       19 | [Vector3int16][Vector3int16]                     | ❌
|       20 | [CFrame][CFrame]                                 | ❌
|       21 | [EnumItem][EnumItem]                             | ❌
|       22 | (reserved)                                       |
|       23 | [NumberSequence][NumberSequence]                 | ✔
|       24 | [NumberSequenceKeypoint][NumberSequenceKeypoint] | ❌
|       25 | [ColorSequence][ColorSequence]                   | ✔
|       26 | [ColorSequenceKeypoint][ColorSequenceKeypoint]   | ❌
|       27 | [NumberRange][NumberRange]                       | ✔
|       28 | [Rect][Rect]                                     | ✔
|       29 | [PhysicalProperties][PhysicalProperties]         | ❌
|       30 | (reserved)                                       |
|       31 | [Region3][Region3]                               | ❌
|       32 | [Region3int16][Region3int16]                     | ❌
| 33 - 255 | (reserved)                                       |

Currently, only a subset of types are officially supported by Roblox, as
indicated above. The formats of unsupported types are subject to change.

## Value types

### String
[String]: #user-content-string

The **String** type corresponds to the `string` Roblox data type.

It is a structure with the following fields:

Field   | Type      | Description
--------|-----------|------------
Length  | `uint32`  | The length of the string, in bytes.
Content | `[]uint8` | The content of the string. The length is determined by the Length field.

### Bool
[Bool]: #user-content-bool

**Type:** `uint8`

The **Bool** type corresponds to the `bool` Roblox data type. 0 represents
false, while other values represent true.

### Int
[Int]: #user-content-int

**Type:** `int32`

The **Int** type corresponds to the `int` Roblox data type.

### Float
[Float]: #user-content-float

**Type:** `float32`

The **Float** type corresponds to the `float` Roblox data type.

### Double
[Double]: #user-content-double

**Type:** `float64`

The **Double** type corresponds to the `double` Roblox data type.

### Array
[Array]: #user-content-array

The **Array** type represents an ordered sequence of values. It is a structure
with the following fields:

Field    | Type               | Description
---------|--------------------|------------
Length   | `uint32`           | The number of elements in the array.
Elements | [`[]Value`][Value] | The elements of the array. The length is determined by the Length field.

### Dictionary
[Dictionary]: #user-content-dictionary

The **Dictionary** type represents a collection of keys mapped to values. It is
a structure with the following fields:

Field   | Type               | Description
--------|--------------------|------------
Length  | `uint32`           | The number of entries in the dictionary.
Entries | [`[]Entry`][Entry] | The entries of the dictionary. The length is determined by the Length field.

### Entry
[Entry]: #user-content-entry

The **Entry** type represents a single key-value pair from a
[Dictionary][Dictionary]. It is a structure with the following fields:

Field | Type               | Description
------|--------------------|------------
Key   | [`String`][String] | The key of the entry.
Value | [`Value`][Value]   | The value of the entry.

### UDim
[UDim]: #user-content-udim

The **UDim** type corresponds to the `UDim` Roblox data type. It is a structure
with the following fields:

Field  | Type      | Description
-------|-----------|------------
Scale  | `float32` | Corresponds to `UDim.Scale`.
Offset | `int32`   | Corresponds to `UDim.Offset`.


### UDim2
[UDim2]: #user-content-udim2

The **UDim2** type corresponds to the `UDim2` Roblox data type. It is a
structure with the following fields:

Field | Type           | Description
------|----------------|------------
X     | [`UDim`][UDim] | Corresponds to `UDim2.X`.
Y     | [`UDim`][UDim] | Corresponds to `UDim2.Y`.

### Ray
[Ray]: #user-content-ray

The **Ray** type corresponds to the `Ray` Roblox data type. It is
a structure with the following fields:

Field     | Type                 | Description
----------|----------------------|------------
Origin    | [`Vector3`][Vector3] | Corresponds to `Ray.Origin`.
Direction | [`Vector3`][Vector3] | Corresponds to `Ray.Direction`.

### Faces
[Faces]: #user-content-faces

**Type:** `uint32`

The **Faces** type corresponds to the `Faces` Roblox data type. The first 6 bits
set the fields of the value. Other bits are ignored.

Field  | Bit | Description
-------|----:|------------
Right  |   0 | Corresponds to `Faces.Right`.
Top    |   1 | Corresponds to `Faces.Top`.
Back   |   2 | Corresponds to `Faces.Back`.
Left   |   3 | Corresponds to `Faces.Left`.
Bottom |   4 | Corresponds to `Faces.Bottom`.
Front  |   5 | Corresponds to `Faces.Front`.

### Axes
[Axes]: #user-content-axes

**Type:** `uint32`

The **Axes** type corresponds to the `Axes` Roblox data type. The first 3 bits
set the fields of the value. Other bits are ignored.

Field | Bit | Description
------|----:|------------
X     |   0 | Corresponds to `Axes.X`.
Y     |   1 | Corresponds to `Axes.Y`.
Z     |   2 | Corresponds to `Axes.Z`.

### BrickColor
[BrickColor]: #user-content-brickcolor

**Type:** `uint32`

The **BrickColor** type corresponds to the `BrickColor` Roblox data type.

### Color3
[Color3]: #user-content-color3

The **Color3** type corresponds to the `Color3` Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
R     | `float32` | Corresponds to `Color3.R`.
G     | `float32` | Corresponds to `Color3.G`.
B     | `float32` | Corresponds to `Color3.B`.

### Vector2
[Vector2]: #user-content-vector2

The **Vector2** type corresponds to the `Vector2` Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
X     | `float32` | Corresponds to `Vector2.X`.
Y     | `float32` | Corresponds to `Vector2.Y`.

### Vector3
[Vector3]: #user-content-vector3

The **Vector3** type corresponds to the `Vector3` Roblox data type. It is a
structure with the following fields:

Field | Type      | Description
------|-----------|------------
X     | `float32` | Corresponds to `Vector3.X`.
Y     | `float32` | Corresponds to `Vector3.Y`.
Z     | `float32` | Corresponds to `Vector3.Z`.

### Vector2int16
[Vector2int16]: #user-content-vector2int16

The **Vector2int16** type corresponds to the `Vector2int16` Roblox data type. It
is a structure with the following fields:

Field | Type    | Description
------|---------|------------
X     | `int16` | Corresponds to `Vector2int16.X`.
Y     | `int16` | Corresponds to `Vector2int16.Y`.

### Vector3int16
[Vector3int16]: #user-content-vector3int16

The **Vector3int16** type corresponds to the `Vector3int16` Roblox data type. It
is a structure with the following fields:

Field | Type    | Description
------|---------|------------
X     | `int16` | Corresponds to `Vector3int16.X`.
Y     | `int16` | Corresponds to `Vector3int16.Y`.
Z     | `int16` | Corresponds to `Vector3int16.Z`.

### CFrame
[CFrame]: #user-content-cframe

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

### EnumItem
[EnumItem]: #user-content-enumitem

The **EnumItem** type corresponds to an item of an Enum. It is a structure with
the following fields:

Field | Type               | Description
------|--------------------|------------
Enum  | [`String`][String] | The name of the enum that this value is an item of.
Value | `uint32`           | The value of the enum item.

### NumberSequence
[NumberSequence]: #user-content-numbersequence

The **NumberSequence** type corresponds to the `NumberSequence` Roblox data
type. It is a structure with the following fields:

Field     | Type                                                 | Description
----------|------------------------------------------------------|------------
Length    | `uint32`                                             | The length of the sequence.
Keypoints | [`[]NumberSequenceKeypoint`][NumberSequenceKeypoint] | The keypoints of the sequence, the length determined by the Length field.

### NumberSequenceKeypoint
[NumberSequenceKeypoint]: #user-content-numbersequencekeypoint

The **NumberSequenceKeypoint** type corresponds to the `NumberSequenceKeypoint`
Roblox data type. It is a structure with the following fields:

Field    | Type      | Description
---------|-----------|------------
Envelope | `float32` | Corresponds to `NumberSequenceKeypoint.Envelope`.
Time     | `float32` | Corresponds to `NumberSequenceKeypoint.Time`.
Value    | `float32` | Corresponds to `NumberSequenceKeypoint.Value`.

### ColorSequence
[ColorSequence]: #user-content-colorsequence

The **ColorSequence** type corresponds to the `ColorSequence` Roblox data type.
It is a structure with the following fields:

Field     | Type                                               | Description
----------|----------------------------------------------------|------------
Length    | `uint32`                                           | The length of the sequence.
Keypoints | [`[]ColorSequenceKeypoint`][ColorSequenceKeypoint] | The keypoints of the sequence, the length determined by the Length field.

### ColorSequenceKeypoint
[ColorSequenceKeypoint]: #user-content-colorsequencekeypoint

The **ColorSequenceKeypoint** type corresponds to the `ColorSequenceKeypoint`
Roblox data type. It is a structure with the following fields:

Field    | Type               | Description
---------|--------------------|------------
Envelope | `float32`          | Corresponds to `ColorSequenceKeypoint.Envelope`.
Time     | `float32`          | Corresponds to `ColorSequenceKeypoint.Time`.
Value    | [`Color3`][Color3] | Corresponds to `ColorSequenceKeypoint.Value`.

### NumberRange
[NumberRange]: #user-content-numberrange

The **NumberRange** type corresponds to the `NumberRange` Roblox data type. It
is a structure with the following fields:

Field | Type      | Description
------|-----------|------------
Min   | `float32` | Corresponds to `NumberRange.Min`.
Max   | `float32` | Corresponds to `NumberRange.Max`.

### Rect
[Rect]: #user-content-rect

The **Rect** type corresponds to the `Rect` Roblox data type. It is a structure
with the following fields:

Field | Type                 | Description
------|----------------------|------------
Min   | [`Vector2`][Vector2] | Corresponds to `Rect.Min`.
Max   | [`Vector2`][Vector2] | Corresponds to `Rect.Max`.

### PhysicalProperties
[PhysicalProperties]: #user-content-physicalproperties

The **PhysicalProperties** type corresponds to the `PhysicalProperties` Roblox
data type. It is a structure with the following fields:

Field            | Type      | Description
-----------------|-----------|------------
CustomPhysics    | `uint8`   | Whether the value has custom physics. False if zero, true if non-zero.
Density          | `float32` | Corresponds to `PhysicalProperties.Density`.
Friction         | `float32` | Corresponds to `PhysicalProperties.Friction`.
Elasticity       | `float32` | Corresponds to `PhysicalProperties.Elasticity`.
FrictionWeight   | `float32` | Corresponds to `PhysicalProperties.FrictionWeight`.
ElasticityWeight | `float32` | Corresponds to `PhysicalProperties.ElasticityWeight`.

### Region3
[Region3]: #user-content-region3

The **Region3** type corresponds to the `Region3` Roblox data type. It is a
structure with the following fields:

Field | Type                 | Description
------|----------------------|------------
Min   | [`Vector3`][Vector3] | The lower bound of the region.
Max   | [`Vector3`][Vector3] | The upper bound of the region.

### Region3int16
[Region3int16]: #user-content-region3int16

The **Region3int16** type corresponds to the `Region3int16` Roblox data type. It
is a structure with the following fields:

Field | Type                           | Description
------|--------------------------------|------------
Min   | [`Vector3int16`][Vector3int16] | The lower bound of the region.
Max   | [`Vector3int16`][Vector3int16] | The upper bound of the region.
