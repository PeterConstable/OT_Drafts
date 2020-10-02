# COLR v1 formats

**Revision 5** 2020-09-03

Incorporates proposed enhancement adding paint format 4 for grouping (analogous to 'glyf' composites), along with Affine2x3.

## Intro
This is a draft of structure specifications for a proposed addition to the OpenType spec.

The format was initially based on a [C++ spec by Google](https://github.com/googlefonts/colr-gradients-spec/blob/master/colr-gradients-spec.md), which has since been updated based on various feedback. A preliminary implementation is reflected in the [noto-handwriting-glyf_colr_1.ttf file](https://github.com/googlefonts/color-fonts/blob/master/fonts/noto-handwriting-glyf_colr_1.ttf) and in [this dump of that font](https://github.com/googlefonts/colr-gradients-spec/files/5064843/noto-handwriting-glyf_colr_1.zip) from a fonttools implementation.

**Note:** this draft may incorporate some proposed revisions not yet be reflected in the above spec, in the fonttools implementation, or in that test font.

## Metric structures

### _VarFWord Record_ (variable design-grid coordinate)

*VarFWord record:*

| Type | Name | Description |
|-|-|-|
| FWORD | coordinate | |
| uint16 | varOuterIndex | |
| uint16 | varInnerIndex | |

(Size: 3 \* 2 = 6 bytes)

### _VarUFWord Record_ (variable design-grid distance)

*VarUFWord record:*

| Type | Name | Description |
|-|-|-|
| UFWORD | distance | |
| uint16 | varOuterIndex | |
| uint16 | varInnerIndex | |

(Size: 3 \* 2 = 6 bytes)

### _VarFixed Record_ (variable fixed value)

*VarFixed record:*

| Type | Name | Description |
|-|-|-|
| Fixed | value | |
| uint16 | varOuterIndex | |
| uint16 | varInnerIndex | |

(Size: 4 + 2 \* 2 = 8 bytes)

* *Note:* In order to combine deltas with Fixed values, the ItemVariationStore format is extended to allow for int32 deltas. When combining a Fixed value with 32-bit deltas, the Fixed value is treated as though it were int32.

### _VarF2Dot14 Record_ (variable, F2Dot14)

*VarF2Dot14 record:*

| Type | Name | Description |
|-|-|-|
| F2Dot14 | value | |
| uint16 | varOuterIndex | |
| uint16 | varInnerIndex | |

(Size: 3 \* 2 = 6 bytes)

* Inherently limited to range [-2., 2.)
* In some contexts, could be limited to range [-1., 1.], or to [0., 1.]
* When combining an F2Dot14 with 16-bit deltas, the F2Dot14 is treated as though it were int16.

### _Affine2x3 Record_ (variable 2×3 matrix)

*Affine2x3 record:*

| Type | Name | Description |
|-|-|-|
| VarFixed | xx | |
| VarFixed | xy | |
| VarFixed | yx | |
| VarFixed | yy | |
| VarFixed | dx | Translation in x direction. |
| VarFixed | dy | Translation in y direction. |

(Size: 6 \* 8 = 48 bytes)

When combining the 2×3 matrix with an (x,y) coordinate pair for a position in the design grid, the coordinate pair is extended to a triple, (x,y,1).

## Color structures

Proposed ```color``` struct has an index to a CPAL ```ColorRecord```, but adds an alpha field that is variable. Note that the CPAL ```ColorRecord``` already has an alpha value. The rationale for adding alpha here is that it is a better design to let a color palette have RGB values only, and to set an alpha / opacity in the elements where the color is used.

### _ColorIndex Record_ (palette index with variable opacity)

*ColorIndex record:*

| Type | Name | Description |
|-|-|-|
| uint16 | paletteIndex | Index for a CPAL palette entry. |
| VarF2Dot14 | alpha | Variable alpha value. |

(Size: 2 + 6 = 8 bytes)

* The alpha.value, and any variations of it, should be in the range [0.0, 1.0] (inclusive); values outside this range should be clipped to the range.

* CPAL palette entries include an alpha value (ColorRecord.alpha). The alpha value from a palette entry must be converted to a value in the range [0., 1.] and multiplied into the ColorIndex.alpha value.

### _ColorStop Record_ (color index with normalized distance)

*ColorStop record:*

| Type | Name | Description |
|-|-|-|
| VarF2Dot14 | stopOffset | Proportional distance on a color line; variable. |
| ColorIndex | color | |

(Size: 6 + 8 = 14 bytes)

* The stopOffset.value, and any variations of it, should be in the range [0., 1.] (inclusive); values outside this range should be clipped to the range.

### _ColorLine Table_

*ColorLine table:*

| Type | Name | Description |
|-|-|-|
| uint8 | extend | An Extend enum value. |
| uint16 | numStops | Number of ColorStop records. |
| ColorStop | colorStops[numStops] | |

(Size: variable; header is 4 bytes)

The extend field must be set to one of the Extend enumeration values:

*Extend enumeration:*

| Value | Name | Description |
|-|-|-|
| 0 | EXTEND_PAD     | Use nearest color stop. |
| 1 | EXTEND_REPEAT  | Repeat from farthest color stop. |
| 2 | EXTEND_REFLECT | Mirror color line from nearest end. |

## Composition modes

Supported composition modes are taken from the W3C [Compositing and Blending Level 1][1] specification.

*CompositeMode enumeration:*

| Value | Name | Description |
|-|-|-|
| | *Porter-Duff modes* | |
| 0 | COMPOSITE_CLEAR | See [Clear][2] |
| 1 | COMPOSITE_SRC | See [Copy][3] |
| 2 | COMPOSITE_DEST | See [Destination][4] |
| 3 | COMPOSITE_SRC_OVER | See [Source Over][5] |
| 4 | COMPOSITE_DEST_OVER | See [Destination Over][6] |
| 5 | COMPOSITE_SRC_IN | See [Source In][7] |
| 6 | COMPOSITE_DEST_IN | See [Destination In][8] |
| 7 | COMPOSITE_SRC_OUT | See [Source Out][9] |
| 8 | COMPOSITE_DEST_OUT | See [Destination Out][10] |
| 9 | COMPOSITE_SRC_ATOP | See [Source Atop][11] |
| 10 | COMPOSITE_DEST_ATOP | See [Destination Atop][12] |
| 11 | COMPOSITE_XOR | See [XOR][13] |
| | *Separable color blend modes:* | |
| 12 | COMPOSITE_SCREEN | See [screen blend mode][14] |
| 13 | COMPOSITE_OVERLAY | See [overlay blend mode][15] |
| 14 | COMPOSITE_DARKEN | See [darken blend mode][16] |
| 15 | COMPOSITE_LIGHTEN | See [lighten blend mode][17] |
| 16 | COMPOSITE_COLOR_DODGE | See [color-dodge blend mode][18] |
| 17 | COMPOSITE_COLOR_BURN | See [color-burn blend mode][19] |
| 18 | COMPOSITE_HARD_LIGHT | See [hard-light blend mode][20] |
| 19 | COMPOSITE_SOFT_LIGHT | See [soft-light blend mode][21] |
| 20 | COMPOSITE_DIFFERENCE | See [difference blend mode][22] |
| 21 | COMPOSITE_EXCLUSION | See [exclusion blend mode][23] |
| 22 | COMPOSITE_MULTIPLY | See [multiply blend mode][24] |
| | *Non-separable color blend modes:* | |
| 23 | COMPOSITE_HSL_HUE | See [hue blend mode][25] |
| 24 | COMPOSITE_HSL_SATURATION | See [saturation blend mode][26] |
| 25 | COMPOSITE_HSL_COLOR | See [color blend mode][27] |
| 26 | COMPOSITE_HSL_LUMINOSITY | See [luminosity blend mode][28] |

## Paint tables

### Paint Format 1: Solid color fill

*PaintSolid table (format 1):*

| Type | Field name | Description |
|-|-|-|
| uint8 | format | Set to 1. |
| ColorIndex | color | Solid color fill. |

(Size: 2 + 8 = 10 bytes)

### Paint Format 2: Linear gradient fill

*PaintLinearGradient table (format 2):*

| Type | Field name | Description |
|-|-|-|
| uint8 | format | Set to 2. |
| Offset24 | colorLineOffset | Offset to ColorLine, from start of PaintLinearGradient table. |
| VarFWord | x0 | Start point x coordinate. |
| VarFWord | y0 | Start point y coordinate. |
| VarFWord | x1 | End point x coordinate. |
| VarFWord | y1 | End point y coordinate. |
| VarFWord | x2 | Rotation vector end point x coordinate. |
| VarFWord | y2 | Rotation vector end point y coordinate. |

(Size (header, excluding ColorLine subtable):  2 + 4 + 3 \* 12 = 42 bytes)

### Paint Format 3: Radial/conic gradient fill

*PaintRadialGradient table (format 3):*

|Type | Field name | Description |
|-|-|-|
| uint16 | format | set to 3 |
| Offset32 | colorLineOffset | offset from start of PaintRadialGradient table |
| VarFWord | x0 | start circle center x coordinate |
| VarFWord | y0 | start circle center y coordinate |
| VarUFWord | radius0 | start circle radius |
| VarFWord | x1 | end circle center x coordinate |
| VarFWord | y1 | end circle center y coordinate |
| VarUFWord | radius1 | end circle radius |
| Offset32 | 2By2TransformOffset | Offset to Affine2By2 table, from start of PaintFormat3 table. May be null. |

(Size (header, excluding ColorLine and Affine2By2 subtables): 2 + 4 + 2 \* 12 + 2 \* 6 + 4 = 46 bytes)

* Defines a class of gradients that are a functional superset of a radial gradient: color gradation along a cylinder defined by two circles.

### Paint Format 4: Glyph outline clip mask

*PaintGlyphOutlineClip table (format 4):*

| Type | Field name | Description |
|-|-|-|
| uint8 | format | Set to 4. |
| Offset24 | paintOffset | Offset to a Paint table, from start of PaintGlyphOutlineClip table. |
| uint16 | glyphID | Glyph ID for the source outline. |

(Size: 6 bytes)

* Glyph outline is used as clip mask for the content in the paint subtable.
* Glyph ID must be less than maxp.numGlyphs

### Paint Format 5: COLR composition

*PaintColrGlyph table (format 5):*

| Type | Field name | Description |
|-|-|-|
| uint8 | format | Set to 5. |
| uint16 | glyphID | Virtual glyph ID for a BaseGlyphV1List base glyph. |

* Glyph ID must be in the BaseGlyphV1List; may be greater than maxp.numGlyphs.

### Paint Format 6: Transformed composition

*PaintTransformed table (format 6):*

| Type | Field name | Description |
|-|-|-|
| uint8 | format | Set to 6. |
| Offset24 | paintOffset | Offset to a Paint subtable, from start of PaintTransform table. |
| Affine2x3 | transform | An Affine2x3 record (inline). |

### Paint Format 7: Composite

*PaintComposite table (format 7):*

| Type | Field name | Description |
|-|-|-|
| uint8 | format | Set to 7. |
| Offset24 | sourcePaintOffset | Offset to a source Paint table, from start of PaintComposite table. |
| uint8 | compositeMode | A CompositeMode enumeration value. |
| Offset24 | backdropPaintOffset | Offset to a backdrop Paint table, from start of PaintComposite table. |

* If compositeMode value is not recognized, COMPOSITE_CLEAR is used.

## COLR Header, base glyph and layer records

### _COLR V1 Header_

*COLR version 1:*

| Type | Field name | Description |
|-|-|-|
| uint16 | version | Table version number—set to 1. |
| uint16 | numBaseGlyphRecords | May be 0 in a version 1 table. |
| Offset32 | baseGlyphRecordsOffset | Offset to baseGlyphRecords array (may be NULL). |
| Offset32 | layerRecordsOffset | Offset to layerRecords array (may be NULL). |
| uint16 | numLayerRecords | May be 0 in a version 1 table. |
| Offset32 | baseGlyphV1ListOffset | Offset to BaseGlyphV1List table. |
| Offset32 | itemVariationStoreOffset | Offset to ItemVariationStore (may be NULL). |

(Size: 4 \* 4 + 3 \* 3 = 22 bytes)

* Offsets are from the start of the COLR table.

### _BaseGlyphV1List_ table

*BaseGlyphV1List table:*

| Type | Name | Description |
|-|-|-|
| uint32 | numBaseGlyphV1Records |  |
| BaseGlyphV1Record | baseGlyphV1Records[numBaseGlyphV1Records] | |

(Size: variable; 4 byte header)

### _BaseGlyphV1Record_

*BaseGlyphV1Record:*

| Type | Name | Description |
|-|-|-|
| uint16 | glyphID | Glyph ID of the base glyph. |
| Offset32 | layerListOffset | Offset to LayerV1List table, from start of BaseGlyphsV1List table. |

(Size: 6 bytes)

### LayerV1List Table

*LayerV1List table:*

| Type | Field name | Description |
|-|-|-|
| uint8 | numLayers |  |
| Offset32 | paintOffset[numLayers] | Offsets to Paint tables, each from the start of the LayerV1List table. |

(Size: variable; 1 byte header)

[1]: https://www.w3.org/TR/compositing-1/
[2]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_clear
[3]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_src
[4]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dst
[5]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_srcover
[6]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dstover
[7]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_srcin
[8]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dstin
[9]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_srcout
[10]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dstout
[11]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_srcatop
[12]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_dstatop
[13]: https://www.w3.org/TR/compositing-1/#porterduffcompositingoperators_xor
[14]: https://www.w3.org/TR/compositing-1/#blendingscreen
[15]: https://www.w3.org/TR/compositing-1/#blendingoverlay
[16]: https://www.w3.org/TR/compositing-1/#blendingdarken
[17]: https://www.w3.org/TR/compositing-1/#blendinglighten
[18]: https://www.w3.org/TR/compositing-1/#blendingcolordodge
[19]: https://www.w3.org/TR/compositing-1/#blendingcolorburn
[20]: https://www.w3.org/TR/compositing-1/#blendinghardlight
[21]: https://www.w3.org/TR/compositing-1/#blendingsoftlight
[22]: https://www.w3.org/TR/compositing-1/#blendingdifference
[23]: https://www.w3.org/TR/compositing-1/#blendingexclusion
[24]: https://www.w3.org/TR/compositing-1/#blendingmultiply
[25]: https://www.w3.org/TR/compositing-1/#blendinghue
[26]: https://www.w3.org/TR/compositing-1/#blendingsaturation
[27]: https://www.w3.org/TR/compositing-1/#blendingcolor
[28]: https://www.w3.org/TR/compositing-1/#blendingluminosity
[29]: https://www.w3.org/TR/compositing-1/#blendingnormal