# Archive File Specification
Specification on how asset files are stored inside of an archive. Include TIL, URK and some DAT files, as used in the DOS version.
## Archive Format
There are several different archive files that all have the same format.\
### Overall File Structure
Archive files consist of two portions, a short header and a block of file contents.\
Header starts at address 0x00 and continues until the first byte of the first file.\
The archives are uncompressed.
### Header Structure
The header consists of 16B entries that specify the 8.3 filename (including the point) and the little-endian 32-bit offset into the archive file where the stored file starts.\
Each 16B block contains:\
0x00 .. 0x0B: The filename, in all-caps ASCII including the point, padded with NUL bytes, e.g. LARGE.DAT\\0\\0\\0.\
0x0C .. 0x0F: 4B int, little-endian offset into the archive file where the stored file starts.
### Data Structure
Files are stored in offset order, so the end of one file is one byte before the start of the next file, or the end of the archive.\
Files are also stored uncompressed, so extracting them is as easy as dumping from the beginning offset to the beginning offset of the next file.
## TIL / URK Files
TIL and URK files effectively the same archive files that contain seven files:

- SMALL.HED
- LARGE.HED
- OTHER.HED
  - Header files for the DOS sprite format
- SMALL.DAT
- LARGE.DAT
- OTHER.DAT
  - Data files for the DOS sprite format
- URKNAME.TXT
  - Information about the archive

## USER.DAT
Contains all of the base assets for the game in various formats.
