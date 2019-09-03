# Sprite File Specification
Specification on how the various sprite data files are formatted for the Windows 95 Special Edition version. Includes [MIF](#mif-files) files.
## LARGE.DAT, SMALLMED.DAT, SPECIAL.DAT
### Overall File Structure
A header and chunks containing a sprite in them pointed to by the header.\
Header starts at address 0x00, and the sprite chunks fill the rest of the file.
### Header Structure
The first two byte contain the number of entries in the header.\
The rest of the header is repeated 10 bytes blocks, with a total number equal to the previous count.\
Each 10B block contains:\
0x00, 0x01: 2B int, the ingame id that the chunk represents.\
0x02 .. 0x05: 4B int,  the absolute offset from the start of the file where the start of a chunk is.\
0x06 .. 0x08: 2B int, the sprite’s height in pixels.\
0x09 .. 0x0A: 2B int, the sprite’s width in pixels.

_Note:_ some blocks with the same ID appear twice, the second it spurious.

### Sprite Data Structure
A sprite is made up of multiple rows of pixel data.\
Each row of pixel data starts with a two 1B ints. The first contains length of the rest of the block after this part. The second indicates whether or not more pixel data follows. 0x01=more, 0x02=last line.\
The amount of pixel data can be 0, which means to draw a blank row.

For a row of pixel data, the structure is as follows:
- 0x00: 1B int, count, varies by mode.
- 0x01: 1B int, the mode that the row can be. Can be 0, 3, or 4.
  - mode 3 (in this mode, pixels start at an offset from the left edge):
	- 0x00: 1B int, number of pixels from the edge to start first pixel.
    - 0x02: 1B int, count of pixels in this row.
    - 0x03: 1B int, extra information (I’m not entirely sure what it does at this time.
    - if the previous two values are both 0:
      - 0x04: 1B int, count of pixels in this row.
      - 0x05: 1B int, extra information (I’m not entirely sure what this does).
      - 0x06 .. count, 1B per pixel, stores the colour information for that pixel.
    - 0x04 .. count: 1B per pixel, stores the colour information for that pixel.
  - mode 4 (in this mode, pixels start right at the edge):
    - 0x00: 1B int, count of pixels in this row.
    - 0x02 .. count, 1B per pixel, stores the colour information for that pixel.
  - mode 0 (in this mode, there are multiple blocks in the row):
    - As in mode 3 (this appears to work, but seems hacky).
	
_Note:_ number of rows = height.

### Colours:
There are three palette files in the SC2k `Bitmap/` directory, that contain a 16x16 grid of pixel colours. The game colours the pixels it displays based on one of these palettes (probably “PAL_MSTR.BMP”) based on the pixel’s number. It does this by taking the high order nibble of the bit, as an integer and uses that as an index for the row in the 16x16 array of colour values, and the low order nibble as the index to the column.
## MIF Files
### Overall File Structure
Is an IFF file, like the .sc2 file format. Generated in SCURK and loaded into the game to change the way buildings looked.\
First 12 bytes are a header that consists of “MIFF”, length of file, “SC2K”.

Two main sections:\
**INFO:** First 4B is count of length, rest (0x72 in length) appears to be information on the file, including where it was saved from and which version of SCURK it was made with, along with some other unknown information.\
**TILE:** First 4B is length of the contents, next two bytes is the number of sub pieces of data, either SHAP or NAME.

#### NAME
Comes before a SHAP object and changes the name displayed for that object.\

|Offset|Purpose|
|---|---|
| 00 .. 03 | length of data following.|
| 04 | Unknown |
| 05 | Building type this name applies to. Same indices as the game uses for XBLD.|
| 06 | Unknown |
| 07 | length of name string|
| 08 .. end | ASCII string with modified name.Maximum string length appears to be 0x16.|

#### SHAP
First 4 bytes is a length.\
After length, structure is:

00 .. 01: building ID\
02 .. 03: width in pixels of tile. Observed width are 8px, 16px, 24px, 32px, 48px, 64px, 96px and 128px\
04 .. 05: height of image in pixels.\
06 .. 0A: length of pixel data.

_Note:_ Pixel data is in same format as stored in sprite data files, with one change. The end of a file in indicated by a row with the form \x02\x01\x02\x02. Other than that, parsing is the exact same, except that sometimes the pixel count in mode 4 is an odd number, and this is likely due to a bug or corruption and should be ignored.

## LARGE.HED, SMALL.HED, OTHER.HED
These are the header files for the DOS version that describe the contents of the LARGE.DAT, SMALL.DAT and OTHER.DAT sprite files. They are always contained in an archive file with an associated DAT file, either the base archive or a TIL/URK file.
### Overall File Structure
Each HED file contains 1500 entries with each entry being 6B + 2B of padding in between.\
Null entries (which do not correspond to a sprite) are all 0xFF, and the padding is always all 0xFF.\

- 0x00 .. 0x03: 4B int, little-endian offse into the DAT file where the sprite begins
- 0x04: 1B int, sprite height
- 0x05: 1B int, sprite width
- 0x06 .. 0x07: 2B padding, always 0xFFFF

## LARGE.DAT, SMALL.DAT, OTHER.DAT
Similarly named to the Windows 95 sprite files, these DOS sprite files have a completely different format. It is always contained in an archive file with an associated HED file, either in the base archive or a TIL/URK file.
### Overall File Structure
DAT files contain sequences of variable length row data. Each sprite starts at an offset specified by the HED file.\
Each row starts with a marker value of 0x10, followed by a 1B int for the number of bytes in the row (this value includes the 1B for the length byte but not the row marker).\
Following the length byte is a byte that specifies the type of the chunk, which have different headers:

- 0x04: This portion of the row starts at the next leftmost pixel. Followed by 1B for the number of pixels in this chunk.
- 0x0C: This portion of the row starts at an offset. Followed by 3B:
	- 1B: The number of pixels for the offset
	- 1B: Unknown
	- 1B: The number of pixels in this chunk

Pixel data follows, which is stored as palette indices, as with the other formats.

Multiple chunks can be stored in the same row to allow for non-adjacent color regions. This is done by specifying a larger number of bytes in the row and putting multiple chunks in a row that don't have enough bytes to fill up the row.

The end of a sprite is marked by 0x00 instead of the 0x10 that marks another row. Some TIL files use the pattern 0x10 0x01 0x00 to mark invalid sprites, as this signifies a sprite with one row with no pixel data in that row.
