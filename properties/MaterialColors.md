# Terrain MaterialColors format
This document describes the binary format of the
[Terrain.MaterialColors][MaterialColors] property.

The format of the MaterialColors property is a constant-sized array of colors.
Each group of 3 bytes represents a Color3 value corresponding to a particular
[Material][Material] enum item. Not all materials are represented. The 3 bytes
in a group correspond to the red, green, and blue components of the Color3,
respectively. The first two groups in the array appear to always be zeroed.

The following table lists the nth byte in the sequence, in decimal format. Each
row corresponds to a particular material, and the columns indicate the red
(`RR`), green (`GG`) or blue (`BB`) component of the material's color:

```text
RR GG BB

00 01 02   (reserved)
03 04 05   (reserved)
06 07 08   Grass
09 10 11   Slate
12 13 14   Concrete
15 16 17   Brick
18 19 20   Sand
21 22 23   WoodPlanks
24 25 26   Rock
27 28 29   Glacier
30 31 32   Snow
33 34 35   Sandstone
36 37 38   Mud
39 40 41   Basalt
42 43 44   Ground
45 46 47   CrackedLava
48 49 50   Asphalt
51 52 53   Cobblestone
54 55 56   Ice
57 58 59   LeafyGrass
60 61 62   Salt
63 64 65   Limestone
66 67 68   Pavement
```

For example, the 34th byte indicates the green component of the Sandstone
material's color.

[MaterialColors]: https://developer.roblox.com/en-us/api-reference/property/Terrain/MaterialColors
[Material]: https://developer.roblox.com/en-us/api-reference/enum/Material
