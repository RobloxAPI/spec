# Specifications
This repository contains unofficial specifications for systems related to
[Roblox][Roblox].

There is a growing desire for third parties to interoperate with Roblox and its
systems. However, at this time, Roblox does not provide documents that specify
their proprietary file formats, protocols, and other related items. The purpose
of this project is to provide a resource for such third parties to produce their
own implementations.

It must be made clear that this project is not official, and is not affiliated
with Roblox Corporation. The documents provided here are not sourced directly
from Roblox, but from third-party reverse-engineering efforts.

[Roblox]: https://corp.roblox.com/

## Categories
Documents are divided into several categories.

### Formats
Documents describing file formats.

Document       | Description
---------------|------------
[rbxl][rbxl]   | Describes the binary format for place and model files. Such files often have the `.rbxl` or `.rbxm` file extensions.
[rbxlx][rbxlx] | Describes the XML format for place and model files. Such files often have the `.rbxlx` or `.rbxmx` file extensions.

[rbxl]: formats/rbxl.md
[rbxlx]: formats/rbxlx.md

### Properties
Certain instance properties are serialized as binary data in a specialized
format. These documents describe such formats.

Document                                   | Description
-------------------------------------------|------------
[AttributesSerialize][AttributesSerialize] | Describes the binary format of the Instance.AttributesSerialize property.
[MaterialColors][MaterialColors]           | Describes the binary format of the Terrain.MaterialColors property.
[Tags][Tags]                               | Describes the binary format of the Instance.Tags property.

[AttributesSerialize]: properties/AttributesSerialize.md
[MaterialColors]: properties/MaterialColors.md
[Tags]: properties/Tags.md
