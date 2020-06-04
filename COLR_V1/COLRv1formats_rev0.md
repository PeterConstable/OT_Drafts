# COLR v1 formats

This is a first draft of structure specifications for a proposed addition to the OpenType spec, based on a [C++ spec by Google](https://github.com/googlefonts/colr-gradients-spec/blob/master/colr-gradients-spec.md).

## Metric structures

### _VarScalar Record_ (variable fixed value)

| Type | Name | Description |
|-|-|-|
| Fixed | scalar | |
| uint16 | varOuterIdx | |
| uint16 | varInnerIdx | |

(Size: 4 + 2 \* 2 = 8 bytes)

### _VarNormalizedScalar Record_ (variable, normalized value)

| Type | Name | Description |
|-|-|-|
| F2Dot14 | scalar | must be in range [-1.0, 1,0] |
| uint16 | varOuterIdx | |
| uint16 | varInnerIdx | |

(Size: 3 \* 2 = 6 bytes)

### _VarDistance Record_ (variable magnitude)

| Type | Name | Description |
|-|-|-|
| UFWORD | distance | |
| uint16 | varOuterIdx | |
| uint16 | varInnerIdx | |

(Size: 3 \* 2 = 6 bytes)

### _VarNormalizedDistance Record_ (variable normalized magnitude)

| Type | Name | Description |
|-|-|-|
| F2Dot14 | distance | must be in range [0., 1.] |
| uint16 | varOuterIdx | |
| uint16 | varInnerIdx | |

(Size: 3 \* 2 = 6 bytes)

### _VarCoordinate Record_ (variable x or y coordinate)

| Type | Name | Description |
|-|-|-|
| FWORD | coordinate | |
| uint16 | varOuterIdx | |
| uint16 | varInnerIdx | |

(Size: 3 \* 2 = 6 bytes)

### _VarPoint Record_ (variable (x,y) coordinate pair)

| Type | Name | Description |
|-|-|-|
| CoordinateRecord | xCoordinate | |
| CoordinateRecord | yCoordinate | |

(Size: 2 \* 6 = 12 bytes)

### _VarAffine2By2_ (variable 2×2 matrix)

| Type | Name | Description |
|-|-|-|
| VarScalarRecord | xx | |
| VarScalarRecord | xy | |
| VarScalarRecord | yx | |
| VarScalarRecord | yy | |

(Size: 4 \* 8 = 32 bytes)

## Color structures

Proposed ```color``` struct is like a CPAL ```ColorRecord``` except that it adds a _second_ alpha / transparency field that is variable.

* Need to call it something other than "color record".

### _ColorIndexVarAlpha Record_ (palette index with variable transparency)

| Type | Name | Description |
|-|-|-|
| uint16 | paletteIndex | |
| VarNormalizedScalar | transparency |  |

(Size: 2 + 6 = 8 bytes)

### _ColorStop Record_ (color index with normalized distance)

Q: Uses normalized scalar ([-1, 1]), or normalized distance ([0, 1])? (Assuming the latter.)

| Type | Name | Description |
|-|-|-|
| VarNormalizedDistance | stopOffset | |
| ColorIndexVarAlpha | color |

(Size: 6 + 8 = 14 bytes)

### _ColorLine Table_

| Type | Name | Description |
|-|-|-|
| uint16 | extend | An extend enum value |
| uint16 | numStops | number of ColorStop records |
| ColorStop | colorStops[_numStops_] | |

(Size: variable; header is 4 bytes)

## Paint tables

### _PaintFormat1 Table_ (solid color)

|Type | Field name | Description |
|-|-|-|
| uint16 | format | set to 1 |
| ColorIndexVarAlpha | color | |

(Size: 2 + 8 = 10 bytes)

### _PaintFormat2 Table_ (linear gradient)

|Type | Field name | Description |
|-|-|-|
| uint16 | format | set to 2 |
| Offset32 | colorLineOffset | offset from start of FillFormat2 table |
| VarPoint | p0 | |
| VarPoint | p1 | |
| VarPoint | p2 | |

(Size (header, excluding ColorLine subtable):  2 + 4 + 3 \* 12 = 42 bytes)

### _PaintFormat3 Table_ (radial gradient)

|Type | Field name | Description |
|-|-|-|
| uint16 | format | set to 2 |
| Offset32 | colorLineOffset | offset from start of FillFormat3 table |
| VarPoint | center0 | |
| VarPoint | center1 | |
| VarDistance | radius0 | |
| VarDistance | radius1 | |
| VarAffine2By2 | transform | |

(Size (header, excluding ColorLine subtable): 2 + 4 + 2 \* 12 + 2 \* 6 + 32 = 74 bytes)

## Header, base glyph and layer records

Questions:

* Why structure for base glyph record with _offset_ to layer record array rather than _index_ into array as in COLR v0? Or rather than base glyph _table_ that contains the layer record array?
  * A table containing the array of records would make this more like existing patterns.
    * For example, ```ScriptList``` (parallel to ```ExtendedBaseGlyph```) _contains_ an array of ```ScriptRecord``` (```ExtendedLayerRecord```) with offsets to ```ScriptTable``` (```Fill```).
    * Also, ```FeatureList```.
* uint32 GIDs? (2 extra bytes in EBG for every color glyph, plus 2 extra bytes for every layer) — at least **6 extra bytes for every color glyph**
* uint32 count of layer records rather than uint16?

### _ExtendedLayer Record_

* Add a flag field? (Could be used for blending options, e.g. "punch" — LP's issue)

|Type | Field name | Description |
|-|-|-|
| uint32 | gID |  |
| Offset32 | paintOffset | offset to Paint table, from start of COLR table |

(Size: 2 \* 4 = 8 bytes)

### _BaseGlyphV1 Record_

|Type | Field name | Description |
|-|-|-|
| uint32 | gID |  |
| uint32 | numLayerV1Records |  |
| Offset32 | layerV1RecordsArrayOffset | offset to layerV1Records array, from start of the COLR table |

(Size: 3 \* 4 = 12 bytes)

### Header

|Type | Field name | Description |
|-|-|-|
| uint16 | version | set to 1 |
| uint16 | numBaseGlyphRecords | may be 0 in a version 1 table |
| Offset32 | baseGlyphRecordsOffset | offset to baseGlyphRecords array, from start of COLR table |
| Offset32 | layerRecordsOffset | offset to layerRecords array, from start of COLR table |
| uint16 | numLayerRecords | may be 0 in a version 1 table |
| uint32 | numBaseGlyphV1Records |  |
| Offset32 | baseGlyphsV1 | offset to baseGlyphV1Records **table**, from start of the CPAL table |
| Offset32 | itemVariationStoreOffset | offset to ItemVariationStore, from start of COLR table |

(Size: 3 \* 2 + 5 \* 4 = 26 bytes)

## Alternate design

This is what's reflected in the noto-handwriting-colr_1.ttf test font.

### _COLR V1 Header_

|Type | Field name | Description |
|-|-|-|
| uint16 | version | set to 1 |
| uint16 | numBaseGlyphRecords | may be 0 in a version 1 table |
| Offset32 | baseGlyphRecordsOffset | offset to baseGlyphRecords array, from start of COLR table |
| Offset32 | layerRecordsOffset | offset to layerRecords array, from start of COLR table |
| uint16 | numLayerRecords | may be 0 in a version 1 table |
| Offset32 | baseGlyphV1ListOffset | offset to baseGlyphV1List **table**, from start of the CPAL table |
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
| Offset32 | layerV1Offset | offset to LayersV1 table, from start of BaseGlyphsV1List table |

(Size: 6 bytes (or 8 if changing to 32-bit GIDs))

### _LayersV1_ Table

|Type | Field name | Description |
|-|-|-|
| uint32 | numPaintRecords |  |
| PaintRecord | paintRecords[numPaintRecords] | |

(Size: variable; 4 byte header)

### _PaintRecord_

|Type | Field name | Description |
|-|-|-|
| uint16 | glyphID | (or could change spec to uint32) |
| Offset32 | paintOffset | offset to Paint table, from start of LayersV1List table |

(Size: 6 bytes (or 8 if changing to 32-bit GIDs))
