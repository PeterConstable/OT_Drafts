# COLR v1 formats

This is a draft of structure specifications for a proposed addition to the OpenType spec.

The format was initially based on a [C++ spec by Google](https://github.com/googlefonts/colr-gradients-spec/blob/master/colr-gradients-spec.md), which has since been updated based on various feedback. A preliminary implementation is reflected in the [noto-handwriting-glyf_colr_1.ttf file](https://github.com/googlefonts/color-fonts/blob/master/fonts/noto-handwriting-glyf_colr_1.ttf) and in [this dump of that font](https://github.com/googlefonts/colr-gradients-spec/files/5064843/noto-handwriting-glyf_colr_1.zip) from a fonttools implementation.

**Note:** this draft may incorporate some proposed revisions not yet be reflected in the above spec, in the fonttools implementation, or in that test font.

## Metric structures

### _VarFixed Record_ (variable fixed value)

>Cut this? (Fixed values aren't really compatible with variation deltas.)

| Type | Name | Description |
|-|-|-|
| Fixed | value | |
| uint16 | varOuterIndex | |
| uint16 | varInnerIndex | |

(Size: 4 + 2 \* 2 = 8 bytes)

### _VarF2Dot14 Record_ (variable, F2Dot14)

| Type | Name | Description |
|-|-|-|
| F2Dot14 | value | |
| uint16 | varOuterIndex | |
| uint16 | varInnerIndex | |

(Size: 3 \* 2 = 6 bytes)

* Inherently limited to range [-2, 2)
* In some contexts, could be limited to range [-1.0, 1.0]
* In some COLR contexts, must be in range [0, 1]

### _VarUFWord Record_ (variable design-grid distance)

| Type | Name | Description |
|-|-|-|
| UFWORD | distance | |
| uint16 | varOuterIndex | |
| uint16 | varInnerIndex | |

(Size: 3 \* 2 = 6 bytes)

### _VarFWord Record_ (variable design-grid coordinate)

| Type | Name | Description |
|-|-|-|
| FWORD | coordinate | |
| uint16 | varOuterIndex | |
| uint16 | varInnerIndex | |

(Size: 3 \* 2 = 6 bytes)

### _Affine2By2_ (variable 2×2 matrix)

| Type | Name | Description |
|-|-|-|
| VarFixed | xx | |
| VarFixed | xy | |
| VarFixed | yx | |
| VarFixed | yy | |

(Size: 4 \* 8 = 32 bytes)

## Color structures

Proposed ```color``` struct has an index to a CPAL ```ColorRecord```, but adds an alpha field that is variable. Note that the CPAL ```ColorRecord``` already has an alpha value. The rationale for adding alpha here is that it is a better design to let a color palette have RGB values only, and to set an alpha / opacity in the elements where the color is used.

### _ColorIndex Record_ (palette index with variable opacity)

| Type | Name | Description |
|-|-|-|
| uint16 | paletteIndex | |
| VarF2Dot14 | alpha | alpha.value, and all variations of it, must be in the range 0 to 1 (inclusive) |

(Size: 2 + 6 = 8 bytes)

### _ColorStop Record_ (color index with normalized distance)

| Type | Name | Description |
|-|-|-|
| VarF2Dot14 | stopOffset | stopOffset.value, and all variations of it, must be in the range 0 to 1 (inclusive) |
| ColorIndex | color |

(Size: 6 + 8 = 14 bytes)

### _ColorLine Table_

| Type | Name | Description |
|-|-|-|
| uint16 | extend | An extend enum value |
| uint16 | numStops | number of ColorStop records |
| ColorStop | colorStops[_numStops_] | |

(Size: variable; header is 4 bytes)

The ```extend``` field must be one of the following values:

| Value | Name | Description |
|-|-|-|
| 0 | EXTEND_PAD     |  |
| 1 | EXTEND_REPEAT  |  |
| 2 | EXTEND_REFLECT |  |

## Paint tables

### _PaintFormat1 Table_ (solid color)

|Type | Field name | Description |
|-|-|-|
| uint16 | format | set to 1 |
| ColorIndex | color | |

(Size: 2 + 8 = 10 bytes)

### _PaintFormat2 Table_ (linear gradient)

|Type | Field name | Description |
|-|-|-|
| uint16 | format | set to 2 |
| Offset32 | colorLineOffset | offset from start of FillFormat2 table |
| VarFWord | x0 | start point x coordinate  |
| VarFWord | y0 | start point y coordinate |
| VarFWord | x1 | end point x coordinate |
| VarFWord | y1 | end point y coordinate |
| VarFWord | x2 | rotation vector end point x coordinate |
| VarFWord | y2 | rotation vector end point y coordinate |

(Size (header, excluding ColorLine subtable):  2 + 4 + 3 \* 12 = 42 bytes)

### _PaintFormat3 Table_ (conical/radial gradient)

|Type | Field name | Description |
|-|-|-|
| uint16 | format | set to 3 |
| Offset32 | colorLineOffset | offset from start of FillFormat3 table |
| VarFWord | x0 | start circle center x coordinate |
| VarFWord | y0 | start circle center y coordinate |
| VarUFWord | radius0 | start circle radius |
| VarFWord | x1 | end circle center x coordinate |
| VarFWord | y1 | end circle center y coordinate |
| VarUFWord | radius1 | end circle radius |
| Offset32 | transformOffset | Offset to Affine2x2 table, from start of PaintFormat3 table. May be null. |

(Size (header, excluding ColorLine and Affix2x2 subtables): 2 + 4 + 2 \* 12 + 2 \* 6 + 4 = 46 bytes)

## COLR Header, base glyph and layer records

### _COLR V1 Header_

|Type | Field name | Description |
|-|-|-|
| uint16 | version | set to 1 |
| uint16 | numBaseGlyphRecords | may be 0 in a version 1 table |
| Offset32 | baseGlyphRecordsOffset | offset to baseGlyphRecords array, from start of COLR table |
| Offset32 | layerRecordsOffset | offset to layerRecords array, from start of COLR table |
| uint16 | numLayerRecords | may be 0 in a version 1 table |
| Offset32 | baseGlyphV1ListOffset | offset to baseGlyphV1List table, from start of the COLR table |
| Offset32 | itemVariationStoreOffset | offset to ItemVariationStore, from start of COLR table |

(Size: 4 \* 4 + 3 \* 3 = 22 bytes)

### _BaseGlyphV1List_ table

|Type | Field name | Description |
|-|-|-|
| uint32 | numBaseGlyphV1Records |  |
| BaseGlyphV1Record | baseGlyphV1Records[numBaseGlyphV1Records] | |

(Size: variable; 4 byte header)

### _BaseGlyphV1Record_

|Type | Field name | Description |
|-|-|-|
| uint16 | glyphID | (or could change spec to uint32) |
| Offset32 | layerListOffset | offset to LayerList table, from start of BaseGlyphsV1List table |

(Size: 6 bytes (or 8 if changing to 32-bit GIDs))

### _LayerList_ Table

|Type | Field name | Description |
|-|-|-|
| uint32 | numLayerV1Records |  |
| LayerV1Record | layerV1Records[numLayerV1Records] | |

(Size: variable; 4 byte header)

### _LayerV1Record_

|Type | Field name | Description |
|-|-|-|
| uint16 | glyphID | (or could change spec to uint32) |
| Offset32 | paintOffset | offset to Paint table, from start of LayerList table |

(Size: 6 bytes (or 8 if changing to 32-bit GIDs))
