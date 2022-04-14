# Instance Tags format
This document describes the binary format of the `Instance.Tags` property.

The format of the Tags property is a list of byte sequences, where each element
is separated by a null character. Each element is interpreted as a Lua string
representing a tag on the instance.

For example, a list with elements "AAA", "BBB", and "CCC" would be encoded as
the following byte sequence:

```text
41 41 41 00 42 42 42 00 43 43 43
```

Elements are meant to be unique, and unordered. Care should be taken to make
sure ordering is consistent, and that duplicates are not present.

*Though extremely rare, files produced by Roblox may contain duplicate
elements.*
