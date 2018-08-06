# Sprite File Specification
Specifations on how the various sprite data files are formatted.
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
0x09 .. 0x0A: 2B int, the sprite’s width in pixels.\

_Note:_ some blocks with the same ID appear twice, the second it spurious.

### Sprite Data Structure
A sprite is made up of multiple rows of pixel data.\
Each row of pixel data starts with a two 1B ints. The first contains length of the rest of the block after this part. The second indicates whether or not more pixel data follows. 0x01=more, 0x02=last line.\
The amount of pixel data can be 0, which means to draw a blank row.\

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
