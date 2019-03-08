# Text File Specifications
Covers the contents of the newspapers in DATA_USA and [other ingame text](#other-ingame-text) in TEXT_USA (though it should work for other localizations, only tested with the USA version). Written for the Windows 95 Special Edition version.
## Newspapers
DATA_USA.DAT has the raw newspaper text, and some metadata on how to build stories.
DATA_USA.IDX has an index for reading the various parts of the newspaper data file.
This specification is incomplete in exactly how this file works, but is enough to figure out what is going on.

### Basic format specification:
DATA_USA.IDX contains offsets for certain sections in the DATA_USA.DAT file. This is used to break segments into specific lengths.

| id | length | purpose |
|---|---|---|
| 0x3e8 | 500 |  |
| 0x3e9 | 500 |  |
| 0x3ea | 10000 |  |
| 0x3eb | 149227 | Actual newspaper text |
| 0x3ec | 160 |  |
| 0x3ed | 160 |  |

Each segment seems to be some amount of data with 0x00 bytes in between values. For segments with strings, 0x00 terminates the string.

### Compression
Uses a simple substitution compression to substitute two letters next to each other (and taking two bytes) with a single byte token. Potentially chosen based on analysis of the input data. Some values are also meant to be substituted with other things, such as they Mayor's name or city name.

#### Token -> value mapping Table:
Note that values in square braces are not something to decompress, but a pointer to fill in a word from a different spot in the data file.

|Value|Output|Value|Output|Value|Output|Value|Output|
|---|---|---|---|---|---|---|---|
| 0x0 | None | 0x40 | @ | 0x80 |  | 0xC0 | di |
| 0x1 | th | 0x41 | A | 0x81 |  | 0xC1 | mo |
| 0x2 | in | 0x42 | B | 0x82 |  | 0xC2 | bu |
| 0x3 | re | 0x43 | C | 0x83 |  | 0xC3 | ho |
| 0x4 | to | 0x44 | D | 0x84 |  | 0xC4 | ea |
| 0x5 | an | 0x45 | E | 0x85 |  | 0xC5 | un |
| 0x6 | at | 0x46 | F | 0x86 |  | 0xC6 |  |
| 0x7 | er | 0x47 | G | 0x87 |  | 0xC7 |  |
| 0x8 | st | 0x48 | H | 0x88 |  | 0xC8 | io |
| 0x9 | en | 0x49 | I | 0x89 |  | 0xC9 | pa |
| 0xA | on | 0x4A | J | 0x8A |  | 0xCA | ra |
| 0xB | of | 0x4B | K | 0x8B |  | 0xCB | ke |
| 0xC | te | 0x4C | L | 0x8C |  | 0xCC | lo |
| 0xD | ed | 0x4D | M | 0x8D |  | 0xCD | ly |
| 0xE | ar | 0x4E | N | 0x8E |  | 0xCE | ri |
| 0xF | is | 0x4F | O | 0x8F |  | 0xCF | sh |
| 0x10 | ng | 0x50 | P | 0x90 |  | 0xD0 |  |
| 0x11 | me | 0x51 | Q | 0x91 |  | 0xD1 |  |
| 0x12 | co | 0x52 | R | 0x92 |  | 0xD2 |  |
| 0x13 | ou | 0x53 | S | 0x93 |  | 0xD3 |  |
| 0x14 | al | 0x54 | T | 0x94 |  | 0xD4 |  |
| 0x15 | ti | 0x55 | U | 0x95 |  | 0xD5 |  |
| 0x16 | es | 0x56 | V | 0x96 |  | 0xD6 |  |
| 0x17 | ll | 0x57 | W | 0x97 |  | 0xD7 |  |
| 0x18 | he | 0x58 | X | 0x98 |  | 0xD8 |  |
| 0x19 | ha | 0x59 | Y | 0x99 |  | 0xD9 | pe |
| 0x1A | it | 0x5A | Z | 0x9A |  | 0xDA | ch |
| 0x1B | ca | 0x5B | [ | 0x9B |  | 0xDB | tr |
| 0x1C | ve | 0x5C | \ | 0x9C |  | 0xDC | ci |
| 0x1D | fo | 0x5D | ] | 0x9D |  | 0xDD | hi |
| 0x1E | de | 0x5E | ^ | 0x9E | le | 0xDE |  |
| 0x1F | be | 0x5F | _ | 0x9F | or | 0xDF | so |
| 0x20 |   | 0x60 | ` | 0xA0 |  | 0xE0 |  |
| 0x21 | ! | 0x61 | a | 0xA1 |  | 0xE1 |  |
| 0x22 | " | 0x62 | b | 0xA2 |  | 0xE2 |  |
| 0x23 | # | 0x63 | c | 0xA3 |  | 0xE3 |  |
| 0x24 | $ | 0x64 | d | 0xA4 |  | 0xE4 |  |
| 0x25 | % | 0x65 | e | 0xA5 |  | 0xE5 |  |
| 0x26 | & | 0x66 | f | 0xA6 |  | 0xE6 |  |
| 0x27 | ' | 0x67 | g | 0xA7 |  | 0xE7 |  |
| 0x28 | ( | 0x68 | h | 0xA8 |  | 0xE8 |  |
| 0x29 | ) | 0x69 | i | 0xA9 | ma | 0xE9 |  |
| 0x2A | * | 0x6A | j | 0xAA | li | 0xEA |  |
| 0x2B | + | 0x6B | k | 0xAB | we | 0xEB |  |
| 0x2C | , | 0x6C | l | 0xAC | ne | 0xEC |  |
| 0x2D | - | 0x6D | m | 0xAD |  | 0xED |  |
| 0x2E | . | 0x6E | n | 0xAE | se | 0xEE | su |
| 0x2F | / | 0x6F | o | 0xAF | nt | 0xEF | rt |
| 0x30 | 0 | 0x70 | p | 0xB0 | wa | 0xF0 | ta |
| 0x31 | 1 | 0x71 | q | 0xB1 | wh | 0xF1 | ge |
| 0x32 | 2 | 0x72 | r | 0xB2 | pr | 0xF2 | rs |
| 0x33 | 3 | 0x73 | s | 0xB3 | si | 0xF3 | ow |
| 0x34 | 4 | 0x74 | t | 0xB4 | as | 0xF4 | us |
| 0x35 | 5 | 0x75 | u | 0xB5 |  | 0xF5 | ss |
| 0x36 | 6 | 0x76 | v | 0xB6 |  | 0xF6 | sp |
| 0x37 | 7 | 0x77 | w | 0xB7 |  | 0xF7 | ac |
| 0x38 | 8 | 0x78 | x | 0xB8 | nd | 0xF8 | il |
| 0x39 | 9 | 0x79 | y | 0xB9 | po | 0xF9 | ic |
| 0x3A | : | 0x7A | z | 0xBA | la | 0xFA | pl |
| 0x3B | ; | 0x7B | { | 0xBB | no | 0xFB | fe |
| 0x3C | < | 0x7C | | | 0xBC | ce | 0xFC | wo |
| 0x3D | =CITYNAME | 0x7D | } | 0xBD | fi | 0xFD | da |
| 0x3E | >TEAMNAME | 0x7E | ~MAYORNAME | 0xBE | yo | 0xFE | ai |
| 0x3F | ? | 0x7F | wi | 0xBF | do | 0xFF | ur |

#### Other mappings

‘--’ in file = ‘-’ in game

[byte seems to be an escape value. This means instead of using the previous lookup table, instead treat it as a pointer to some other piece of data. See Lookup table below.

[0x20: looks to be the literal ‘& ‘ characters (that’s a space there).

%x is some number. X seems to be the maximum the number will be, or not. %100=1200 in one article. %1000 = 7000, %20 = 30,370, %50=360, %250=1350, %4=30 %250,000=2350,000

^ seems to behave similarly. ^x97 seems to be a country name just like [0x97. Could be a second name.

@ also @0xa4 seems to be a foreign last name. Just link [0xa4.

\* also seems to behave similarly.

**Escape Value Table:**\
Pointer is a friendly name for the thing being pointed to that was created for the purposes of this documentation.

_Note:_ There is likely some index in one of the other parts of the data file for how to build these, but it has not been deciphered yet.

|Value|Pointer|Value|Pointer|Value|Pointer|Value|Pointer|
|---|---|---|---|---|---|---|---|
| 0x4B | foundingreaction | 0x77 | utterance | 0x95 | action1 | 0xB3 | illness |
| 0x4D | reaction | 0x78 | feeling | 0x96 | action2 | 0xB4 | amount |
| 0x4E | missimanswer | 0x79 | scareverb | 0x97 | country | 0xB5 | importantperson |
| 0x4F | gripeclosing | 0x7A | violentverb | 0x98 | criminal | 0xB6 | moneything |
| 0x55 | gripe1 | 0x7B | pastviolentverb | 0x99 | crime | 0xB7 | moneyevent |
| 0x56 | gripe2 | 0x7C | verb1 | 0x9A | infrastructure | 0xB8 | money |
| 0x57 | gripe3 | 0x7D | verb2 | 0x9B | direction | 0xB9 | month |
| 0x58 | gripe4 | 0x7E | presemtverb1 | 0x9C | illness2 | 0xBA | positive |
| 0x59 | gripe5 | 0x7F | help | 0x9D | exclamation | 0xBB | numberword |
| 0x5A | gripe6 | 0x80 | pasthelp | 0x9E | expert | 0xBC | noun |
| 0x5B | gripe7 | 0x81 | negativeverb | 0x9F | emotion | 0xBD | job |
| 0x5C | traffic | 0x82 | pastverb | 0xA0 | compete | 0xBE | territory |
| 0x5D | pollution | 0x83 | thingdescriptor | 0xA1 | malename | 0xBF | powertype |
| 0x5E | crime | 0x84 | adjective | 0xA2 | femalename | 0xC0 | relative |
| 0x5F | taxes | 0x85 | adverb1 | 0xA3 | foreignfirstname | 0xC1 | smalladjective |
| 0x60 | employment | 0x86 | adverb2 | 0xA4 | foreignlastname | 0xC2 | sport |
| 0x61 | education | 0x87 | time | 0xA5 | 2xforeignname | 0xC3 | injury |
| 0x62 | health | 0x88 | negativefeeling | 0xA6 | name | 0xC4 | sportscore |
| 0x6B | destroyverb | 0x89 | animal | 0xA7 | NGO | 0xC5 | teamname |
| 0x6C | creationverb | 0x8A | negativeadjective | 0xA8 | room | 0xC6 | business |
| 0x6D | pastcreationverb | 0x8B | bodypart | 0xA9 | numericposition | 0xC7 | location |
| 0x6E | fireverb | 0x8C | disaster | 0xAA | invention | 0xC8 | research |
| 0x6F | increase | 0x8D | bignumber | 0xAB | invetion2 | 0xC9 | badthing |
| 0x70 | verb1 | 0x8E | locals | 0xAC | politicalissue | 0xCA | waterbody |
| 0x71 | verb2 | 0x8F | localgov | 0xAD | product | 0xCB | group |
| 0x72 | futureverb | 0x90 | landmark | 0xAE | size | 0xCC | governmentthing |
| 0x73 | makeverb | 0x91 | othercity | 0xAF | title | 0xCD | weathercondition |
| 0x74 | pastmakeverb | 0x92 | localcity | 0xB0 | lname | 0xCE | weekday |
| 0x75 | makenoun | 0x93 | response3 | 0xB1 | legalthing |  |  |
| 0x76 | response | 0x94 | shortsaying | 0xB2 | llama |  |  |

### How Newspaper Stories Are Created

To be determined.

## Other Ingame Text

Stored in TEXT_USA.DAT with an index in TEXT_USA.IDX

Format is relatively straightforward:\
The index file contains an ID for the text (likely to identify it through some in-game mechanism) and a length of the ASCII text block.\
Parsing the data file just involves splitting it up into chunks with the given lengths and converting the RAW bytes to ASCII characters. Note that there seems to be some characters outside the ASCII range, ignoring them doesn't seem to cause any ill effects in parsing.


