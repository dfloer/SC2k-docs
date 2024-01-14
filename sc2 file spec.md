# SimCity 2000 Windows 95 .sc2 File Specification

## Basic File Layout

The basic file structure is of an EA IFF file: \
It starts with a 12B (byte) header which contains:
- First 4B is the IFF type, for .sc2 files this is FORM.
- Next 4B is the length of the file (not counting the first 8B)
- Last 4B is a container for the rest of the file, for .sc2 files this is SCDH.

## Other file notes
For anything on a grid in the save (most segments), the grid goes from top corner (default rotation) to bottom, row by row right to left.

This is an unofficial specification, and is therefore incomplete. However, large sections are complete. This specification also only applies to the .sc2 file format, and not how the game interprets the given values.

Complete sections:\
CNAM, XTER, XBLD, XZON, XUND, XBIT, XTRF, XPLT, XVAL, XCRM, XPLC, XFIR, XPOP, XROG, XGRP, TEXT

Mostly complete sections:\
MISC, XTXT, XLAB, XMIC, SCEN

Sections that need more work:\
XTHG, PICT

## Chunks:

Rest of the file has a 4B chunk id (which is 4 ASCII characters between 0x20 and 0x7F inclusive), a 4B integer size for the chunk and finally the chunk data of the same length, in bytes, as the size of the compressed chunk.

#### Chunks in the file and their uncompressed lengths:

- [CNAM](#cnam): 32 (Stored uncompressed. May be optional in Mac files.)
- [MISC](#misc): 4800
- [ALTM](#altm): 32768 (Stored uncompressed.)
- [XTER](#xter): 16384
- [XBLD](#xbld): 16384
- [XZON](#xzon): 16384
- [XUND](#xund): 16384
- [XTXT](#xtxt): 16384
- [XLAB](#xlab): 6400
- [XMIC](#xmic): 1200
- [XTHG](#xthg): 480
- [XBIT](#xbit): 16384
- [XTRF](#xtrf): 4096
- [XPLT](#xplt): 4096
- [XVAL](#xval): 4096
- [XCRM](#xcrm): 4096
- [XPLC](#xplc): 1024
- [XFIR](#xfir): 1024
- [XPOP](#xpop): 1024
- [XROG](#xrog): 1024
- [XGRP](#xgrp): 3328

#### Chunks only in Scenario Files

Scenario specific entries are uncompressed.

- [TEXT](#text): variable length (2 entries)
- [SCEN](#scen): 56
- [PICT](#pict): variable length
- [TMPL](#tmpl): 338

## Compression:

A variant of Run-Length Encoding. Comes in two variants, a 1 + n byte version and a two byte version.
1. 1 + n byte version: first byte is in range [0 .. 127] means that there are n bytes of uncompressed data corresponding to the byte’s value after it.
2. 2 byte version: first byte is in range [129 .. 255] means to repeat the next byte n times (the RLE part), where (byte value) - 127 = n.

Note that this encoding scheme can lead to certain sections being larger than if they were uncompressed if they’re very random or alternating data. \
It appears that every byte is compressed, even if it’s a single byte on its own. There are lots of 0x01 followed by single bytes. Runs are compressed as normal.

## Chunk Data Format:

### CNAM

Contains 0x1F as its first byte and then 31 bytes after that. The first ASCII range bytes are the name, but the rest could be garbage data. \
_Notes:_ Maybe 0x1F (31) is supposed to be the length of the title after it. There appears to be garbage in it though, but the name is terminated with 0x00. If no valid name (starts with 0x00), game uses all-caps filename as city name. This looks a lot like an unsafe memory copy without checking the length of memory copied for the cstring (null terminated) in the original game code.

### MISC

Miscellaneous city data, 4B/32b integers:

| Offset | Name | Notes|
|---|---|---|
|0000| Unknown (header?) | `00 00 01 22` in 114 cities checked.|
|0004 | City Mode | 0 for terrain edit mode, 1 for city, 2 for disaster mode.|
|0008 | Rotation | Internally called compass. Number between 0 and 3 corresponding to number of counter-clockwise rotations.|
|000C | Year Founded | Year city was founded.|
|0010 | City Age | days since city was founded in 300 day years, and 25 day months? example: date=2435, founded= 2050, (2435-2050)=385. month=July=07, so 385(300)+7*25=115,650. In save file: 115,663, so probably a few day into July.|
|0014 | Money | Stored as signed 32b int |
|0018 | Number of bonds. |
|001C |  Game Level | Game difficulty. 0 = None (Map), 1 = Easy, 2 = Medium, 3 = Hard. |
|0020 |  City Status | Reward tier obtained. 0 = None, 1 = Mayor's Mansion, 2 = City Hall, 3 = Statue, 4 = Military, 5 = Llama Dome, 6 = Arcos. |
|0024 | City Value | Unknown exactly, stores something to do with total city value.|
|0028 | Land Value | Sum of all the values in XVAL. |
|002C | Crime Count | This is a sum of all of the values stored in XCRM.|
|0030 | Traffic Count | Something to do with traffic, **not** a sum of all value sin XTRF. |
|0034 | Pollution | Unknown |
|0038 | City Fame | Unknown |
|003C | Advertising | Unknown |
|0040 | Garbage | Unknown |
|0044 | Work Force % | What percentage of the population is working? |
|0048 | Work Force LE | LE = Life Expectancy |
|004C | Work Force EQ | EQ = Education Quotient |
|0050| National Population | Population of SimNation|
|0054 |  National Value| Unknown |
|0058 | National Tax | Unknown |
|005C| National Trend | How is the sim nation economy doing? This affects interest rates. |
|0060| Heat | Something weather related.|
|0064 | Wind | Something weather related. |
|0068 | Humid | Something weather related. |
|006C | Weather | Called weatherTrend by the game. See [Weather Type table](#weather-type).|
|0070 | Disasters | See [Disaster Type table](#disaster-type).|
|0074 | Residential population| Unknown exactly.|
|0078 | Rewards availability | (0s)11111= all 5, 00001=mayor’s only, 10000=arcos only, etc.|
|007C .. 0168 | Population/Health/Education Graph Data| 20 values each, interleaved in that order.|
|016C .. 01EC | Industry Graph Data | 33x4B. Each appears to be for the industry window. There are 3 different graphs, each of which appears to show be stored as ratios, tax rate, demand. _Ranges on values unknown._|
|01F0 .. 05EC | Building Tile Counts | Same index as XBLD, count of that # of building tiles in the city.|
|05F0 | Populated Tile Count | Total number of populated tiles.|
|05F4 | ? | Unknown, but seems like it'd be the other part of Populated Tile count |
|05F8 | Residential Tile Count | |
|05FC | ? | Something like it should be the other half of Residential Tile Count.|
|0600 | Commercial Tile Count | |
|0604 | ? | Something like the other half of commercial tile count.|
|0608 | Industrial Tile Count | |
|060C | ? | Something like the other half of industrial tile count.|
|0610 .. 06D7 | Bond Data | bonds, maximum of 50, signed 32b int. Famous overflow in game. |
|06D8 .. 0718 | Neighbour Data | 4x4B of neighbour information. Form is: neighbour index, neighbour population, neighbour value (unknown what exactly this is) and neighbour fame (again unknown). Ordering is: lower left, upper left,  unknown,  upper right,  bottom right.|
|0710 | Unknown | Seems to be 0 in an established city (full RCI), +’ve in others. Unknown exactly.|
|0718 .. 0720 | RCI demand | Signed 32b int from -2000 to +2000. First R, second C, third I.|
|0738 .. 0778 | Technology Discovery Years | Contains the year the technology was discovered. Appears to be 0 if the city was saved after the technology was invented. Details in [Technology Discovery Years Table](#technology-discovery-years)|
|077C .. 08BF | Property taxes | Each takes 27*4B. See [Tax Rate Table](#budget-tax-rate-details) |
| 077C | residential tax rate | Residential zoned building tax rate, between 0 and 20. |
| 07E8 | commercial tax rate | Commercial zoned building tax rate, between 0 and 20. |
| 0854 | Industrial tax rate | Industrial zoned building tax rate, between 0 and 20. |
| 08C0 .. 092C | Ordinances budget window information | Contents follow the same 27x4B format, but exact contents unknown at this time |
| 0930 .. 0994 | Bond budget window information | Stores information for the budget window. Details in section [Bond Details](#bond-details).
| 0998 .. .0E38 | City services from the budget panel | See [Budget City Service Details](#budget-city-service-details) section.|
| 0E3C | Year end | Unknown |
| 0E40 | Water level | Water table level.|
| 0E44 | terrain - coast | Was this city generated with coastal terrain. |
| 0E48 | terrain - river | Was this city generated with river terrain. |
| 0E4C | Military Base | Not offered base: 0, offered base but no suitable location/refused base: 1, 2: army base, 3: airbase, 4 navy: base, 5: missile silos |
| 0E50 .. 0EC4 |Newspaper List | Unknown exactly, 6x5B structure.|
| 0EC8 .. 0EFC | Newspaper List | Unknown exactly, 9x6B structure.|
| 0FA0 | Ordinances flags | Bit flags for which of the 20 ordinances are enacted. 00000000:none, 000fffff:20. First ordinances (finance) section are rightmost bits. |
| 0FA4 | unemployed | Unknown |
| 0FA8 .. 0FE4 | Military Count | 16x2B count of military tiles. See [Military Tile Count](#military-tile-count-details) section. |
| 0FE8 | Subway Count | Unknown |
| 0FEC | Speed | Speed setting: 1=paused, 2=Turtle, 3=Llama, 4=Cheetah, 5=African Swallow |
| 0FF0 | Auto Budget | Auto budget setting. |
| 0FF4 | Auto Goto | Auto goto setting. |
| 0FF8 | Sound | Sound effects setting. |
| 0FFC | Music | Music setting. |
| 1000 | Disasters | No disasters setting. |
| 1004 | Paper Delivery | Is paper delivery enabled. |
| 1008 | Extra Newspaper | Is the Extra!!! newspaper enabled. |
| 100C | Newspaper Choice | Which newspaper is chosen to be delivered.|
| 1010 | Unknown | Observed to be 0x80 in many cities. |
| 1014 | Seems to have something to do with zoom and position of map. |
| 1018 | View X | X coordinates for the center of the view. |
| 101C | View Y | Y coordinates for the center of the view. |
| 1020 | Arco Population | Total city population from arcos. |
| 1024 | Connection Tiles | Appears to be a count of tiles that are connected to neighbours.|
| 1028 | Sports Teams | Count of active sports teams from stadiums.|
| 102C | Normal Population | Total city population from normal zones (not arcos). |
| 1030 | Industry Bonus | Unknown |
| 1034 | Pollution Bonus | Unknown |
| 1038 | Old Arrest | Sum of all the police station microsim arrests. |
| 103C | Police Bonus | Unknown |
| 1040 | Disaster  | Unknown |
| 1044 | Unknown | Unknown |
| 1048 | Disaster Active | Seems to be 1 if there’s a disaster happening, 0 otherwise. |
| 104C | Go Disaster | Unknown |
| 1050 | Sewer Bonus | Unknown, game doesn't have sewers. Perhaps another name for the water pipes? |
| 1054 .. 10B4 | All Zero Bytes | Observed in all cities checked. |
| 10B8 | Unknown | small (~-15000) negative number or small positive (~20) signed 32b int. Does not appear in most cities checked.|
| 10BC - 12BC | All Zero Bytes | Observed in all cities checked. |

### MISC Tables

#### Weather Type

|Value | Type|
|---|---|
|00 | Cold|
|01 | Clear|
|02 | Hot|
|03 | Foggy|
|04 | Chilly|
|05 | Overcast|
|06 | Snow|
|07 | Rain|
|08 | Windy|
|09 | Blizzard|
|0A | Hurricane|
|0B | Tornado|

#### Disaster Type

|Value | Type|
|---|---|
|0x0 | none|
|0x1 | fire|
|0x2 | flood|
|0x3 | riot|
|0x4 | toxic spill|
|0x5 | buggy air crash|
|0x6 | quake|
|0x7 | tornado|
|0x8 | monster|
|0x9 | meltdown|
|0xA | microwave|
|0xB | volcano|
|0xC | firestorm|
|0xD | mass riots|
|0xE | mass floods|
|0xF | pollution accident|
|0x10 | hurricane|
|0x11 | buggy helicopter crash|
|0x12 | plane crash|

#### Technology Discovery Years
|Offset|Technology |Notes|
|---|---|---|
|0738| Gas power | |
|073C | Nuclear power | |
|0740 | Solar power | |
|0744 | Wind power | |
|0748 | Microwave power | |
|074C | Fusion power | |
|0750 | Airport | |
|0754 | Highways | |
|0758 | Buses | |
|075C | Subways | |
|0760 | Water treatment | |
|0764 | Desalinization | |
|0768 | Plymouth arco | |
|076C | Forest arco | |
|0770 | Darco | |
|0774 | Launch Arco | |
|0778 | Highways | Odd, but observed in a few cities.|

#### Budget Tax Rate Details
Population is total population of occupied tiles being taxed.
The number corresponds to a 4B offset for the start of the segment.

0. Current population. (Maybe)
1. current tax rate (0 .. 20%)
2. unknown
3. January population
4. tax rate January
5. February population
6. tax rate February
7. March population
8. tax rate March
9. April population
10. tax rate April
11. May population
12. tax rate May
13. June population
14. tax rate June
15. July population
16. tax rate July
17. August population
18. tax rate August
19. September population
20. tax rate September
21. October population
22. tax rate October
23. November population
24. tax rate November
25. December population
26. tax rate December

#### Bond Details

The rate for the bond is stored as an integer, multiplied by 10,000. Example: a rate of 4% is stored as the integer 40,000.

The average rate is the arithmetic average of all of the bonds currently floated, while the sum rate is just the sum of all the rates. Example: A 4% bond and a 25% bond have an average rate of 14.5%, with a sum rate of 29%.

Note: In cities that have used the negative FUND trick/cheat, the "Sum Bond Rate" is incorrect. This is likely an artifact of the bug that allows the trick to work, and doesn't seem to matter.

1. Current Count of Bonds Floated
2. Current Average Bond Rate
3. Sum Bond Rate
4. January Count
5. January Average Rate
6. February Count
7. February Average Rate
8. March Count
9. March Average Rate
10. April Count
11. April Average Rate
12. May Count
13. May Average Rate
14. June Count
15. June Average Rate
16. July Count
17. July Average Rate
18. August Count
19. August Average Rate
20. September Count
21. September Average Rate
22. October Count
23. October Average Rate
24. November Count
25. November Average Rate
26. December Count
27. December Average Rate

#### Budget City Service Details

Each segment has 27 x 4B entries structured.
The number corresponds to a 4B offset for the start of the segment.
0. current number of that building.
1. current funding rate (0 .. 100%)
2. unknown
3. January count of building
4. funding % for January
5. February count of building
6. funding % for February
7. March count of building
8. funding % for March
9. April count of building
10. funding % for April
11. May count of building
12. funding % for May
13. June count of building
14. funding % for June
15. July count of building
16. funding % for July
17. August count of building
18. funding % for August
19. September count of building
20. funding % for September
21. October count of building
22. funding % for October
23. November  count of building
24. funding % for November
25. December count of building
26. funding % for December

Starting offset for section (relative to start of MISC segment): \
_Note:_ Bus stations have no funding setting in the game and therefore aren't stored in the saved city either.

|Offset|Section Type|
|---|---|
| 0x0998 | police funding rate |
| 0x0A04 | fire funding rate |
| 0x0A70 | health funding rate |
| 0x0ADC | education funding rate, Schools |
| 0x0B48 | education funding rate, Colleges |
| 0x0BB4 | transit funding rate, Roads |
| 0x0C20 | transit funding rate, Highways |
| 0x0C8C | transit funding rate, Bridges |
| 0x0CF8 | transit funding rate, Rail |
| 0x0D64 | transit funding rate, Subway |
| 0x0DD0 | transit funding rate, Tunnel |

#### Military Tile Count Details

| Index | Tile ID | Tile Name |
|---|---|---|
| 0 | None | Unknown. Total count of military tiles? |
| 1 | 0xDD | Straight Runway |
| 2 | 0xDE | Cross Runway |
| 3 | 0xEF | Military Parking Lot |
| 4 | 0xF2 | Cargo Yard |
| 5 | 0xEA | Radar |
| 6 | 0xE3 | Warehouse |
| 7 | 0xE4 | Airport Building 1 |
| 8 | 0xE5 | Airport Building 2 |
| 9 | 0xF1 | Top Secret |
| 10 | 0xE0 | Crane |
| 11 | 0xE2 | Military Control Tower |
| 12 | 0xE7 | F-15B |
| 13 | 0xE8 | Small Hangar |
| 14 | 0xE8 | Large Hangar |
| 15 | 0xF9 | Missile Silo |

## ALTM

Altitude map of the city. Stores the altitude of a tile.

2 bytes per tile, stored as a 16 bit integer:\
Bits 0 to 5: This may be related to tunnel levels. _Needs further investigation._\
Bits 6 to 10: 5 bit water level height (32 levels) \
Bits 11 to 15: 5 bit land altitude - for underwater tiles, this is the sea floor (32 levels)

## XTER

Describes how a terrain tile slopes, based on its 4 corners and is represented as 1 byte per tile.

a is top left:\
|a|b|\
|x|y|

Tile type denoted as: 0=down, 1=up\
Given as value: tile type\
**Dry Land:**\
00: 0000\
01: 1100\
02: 0101\
03: 0011\
04: 1010\
05: 1101\
06: 0111\
07: 1011\
08: 1110\
09: 0100\
0A: 0001\
0B: 0010\
0C: 1000\
0D: 1111\
0E .. 0F: unknown. There aren't any other possible sprites in the game files.

**Underwater:**\
10 .. 1D: Same as for dry land, but underwater.\
1E .. 1F: unknown

**Shoreline:**\
20 .. 2D: Same as for dry land, 1 = up, or dry land, 0 = down or in the water.\
2E .. 2F: unknown

**Surface Water:**\
30 .. 3D: Same as for dry land and shoreline, just with surface (0 depth water).\
3E: Waterfall tiles (such as under dams and elsewhere).\
3F: unknown

**More surface water:**\
40: LR stream\
41: TB stream\
42: bay, water Bottom\
43: bay, water Left\
44: bay, water Top\
45: bay, water Right\
46 .. FF: unknown\
_Note: none of the above unknown values appeared in multiple cities._

## XBLD
Stores what building occupies a tile.
Index: Name (SCURK name if different)

**Ground Cover:**\
00: Clear Ground\
01: Rubble (Rubble 1)\
02: Rubble (Rubble 2)\
03: Rubble (Rubble 3)\
04: Rubble (Rubble 4)\
05: Radioactive Waste (Radioactivity)\
06: Trees (Tree)\
07: Trees (Couple O Trees)\
08: Trees (More Trees)\
09: Trees (Morer Trees)\
0A: Trees (Even More Trees)\
0B: Trees (Tons O Trees)\
0C: Trees (Veritable Jungle)\
0D: Small park

**Power Lines:**\
(L = Left, R = Right, T = Top, B = Bottom, H = High: top of slope)\
0E: L-R\
0F: T-B\
10: HT-B\
11: L-HR\
12: T-HB\
13: HL-R\
14: BR\
15: BL\
16: TL\
17: TR\
18: RTB\
19: LBR\
1A: TLB\
1B: LTR\
1C: LTBR

**Roads:**\
1D: L-R\
1E: T-B\
1f: HT-B\
20: L-HR\
21: T-HB\
22: HL-R\
23: BR\
24: BL\
25: TL\
26: TR\
27: RTB\
28: LBR\
29: TLB\
2A: LTR\
2B: LTBR

**Rail:**\
Additional slope pieces denoted with H for the half-high end.\
2C .. 3A: Coding as for power lines\
3B: HT-B\
3C: L-HR\
3D: T-HB\
3E: HL-R

**Tunnel Entrances:**\
3F: T\
40 :R\
41: B\
42: L

**Crossovers:**\
43: Power-TB, Road-LR\
44: Power-LR, Road-TB\
45: Road-LR, Rail-TB\
46: Road-TB, Rail-LR\
47: Power-TB, Rail-LR\
48: Power-LR, Rail-TB

**Highways:**\
49: LR\
4A: TB

**Highway Crossovers:**\
4B: LR, Road-TB\
4C: TB, Road-LR\
4D: LR, Rail-TB\
4E: TB, Rail-LR\
4F: LR, Power-TB\
50: TB, Power-LR

**Bridges:**\
51: Suspension bridge start B\
52: Suspension bridge middle B\
53: Suspension bridge center\
54: Suspension bridge middle T\
55: Suspension bridge end T\
56: Raising bridge tower\
57: Causeway pylon\
58: Raising bridge middle (lowered)\
59: Raising bridge middle (raised)\
5A: Rail bridge, pylon\
5B: Rail bridge\
5C: Elevated Power Lines

**Highway Entrance (Onramps):**\
_Note:_ These can be rotated 90 degrees by bit 6 in XBIT.\
5D: Highway-T, Road-L\
5E: Highway-T, Road-R\
5F: Highway-B, Road-L\
60: Highway-B, Road-R

**Highway bits:**\
61: HT-B\
62: L-HR\
63: T-HB\
64: HL-R\
65: BR\
66: BL\
67: TL\
68: TR\
69: LTBR

**Reinforced Bridge:**\
6A: Pylon\
6B: No pylon

**Subway to Rail:**\
6C: Subway-T\
6D: Subway-R\
6E: Subway-B\
6F: Subway-L

**Residential 1x1:**\
70: Lower-class homes (Lower Class Homes 1)\
71: Lower-class homes (Lower Class Homes 2)\
72: Lower-class homes (Lower Class Homes 3)\
73: Lower-class homes (Lower Class Homes 4)\
74: Middle-class homes (Middle Class Homes 1)\
75: Middle-class homes (Middle Class Homes 2)\
76:  Middle-class homes (Middle Class Homes 3)\
77: Middle-class homes (Middle Class Homes 4)\
78: Luxury Homes (Upper Class Homes 1)\
79: Luxury Homes (Upper Class Homes 2)\
7A: Luxury Homes (Upper Class Homes 3)\
7B: Luxury Homes (Upper Class Homes 4)

**Commercial 1x1:**\
7C: Gas Station (Gas Station 1)\
7D: Bed & Breakfast Inn (Bed and Breakfast Inn)\
7E: Convenience store (Convenience Store)\
7F: Gas Station (Gas Station 2)\
80: Small office building (Small Office Building 1)\
81: Office Building (Small Office Building 2)\
82: Warehouse (Warehouse)\
83: Cassidy’s Toy Store (Cassidy’s Toy Store)

**Industrial 1x1:**\
84: Warehouse (Small WareHouse 1)\
85: Chemical Storage (Chemical Storage)\
86: Warehouse (Small WareHouse 1)\
87: Industrial substation (Industral Substation)

**Misc 1x1:**\
88: Construction (Construction 7)\
89: Construction (Construction 8)\
8A: Abandoned building (Abandoned Building 1)\
8B: Abandoned building (Abandoned Building 2)

**Residential 2x2:**\
8C: Cheap apartments (Small Apartments 1)\
8D: Apartments (Small Apartments 2)\
8E: Apartments (Small Apartments 3)\
8F: Nice Apartments (Medium Apartments 1)\
90: Nice Apartments (Medium Apartments 2)\
91: Condominium (Medium Condominiums 1)\
92: Condominium (Medium Condominiums 2)\
93: Condominium (Medium Condominiums 3)

**Commercial 2x2:**\
94: Shopping center (Shopping Center)\
95: Grocery store (Grocery Store)\
96:  Office Building (Medium Office Building 1)\
97: Resort hotel\
98: Office Building (Medium Office Building 2)\
99: Office/Retail (Office/Retail)\
9A:  Office Building (Medium Office Building 3)\
9B:  Office Building (Medium Office Building 4)\
9C:  Office Building (Medium Office Building 5)\
9D: Office Building (Medium Office Building 6)

**Industrial 2x2:**\
9E: Warehouse (Medium Warehouse)\
9F: Chemical Processing (Chemical Processing 2)\
A0: Factory (Small Factory 1)\
A1: Factory (Small Factory 2)\
A2: Factory (Small Factory 3)\
A3: Factory (Small Factory 4)\
A4: Factory (Small Factory 5)\
A5: Factory (Small Factory 6)

**Misc 2x2:**\
A6: Construction (Construction 3)\
A7: Construction (Construction 4)\
A8: Construction (Construction 5)\
A9: Construction (Construction 6)\
AA: Abandoned building (Abandoned Building 7)\
AB: Abandoned building (Abandoned Building 7)\
AC: Abandoned building (Abandoned Building 7)\
AD: Abandoned building (Abandoned Building 7)

**Residential 3x3:**\
AE: Large apartment building (Large Apartments 1)\
AF: Large apartment building (Large Apartments 2)\
B0: Condominium (Large Condominiums 1)\
B1: Condominium (Large Condominiums 2)

**Commercial 3x3:**\
B2: Office park (Office Park)\
B3: Office tower (Office Tower 1)\
B4: Mini-mall (Mini Mall)\
B5: Theater square (Theatre)\
B6: Drive-in theater (Drive In)\
B7: Office tower (Office Tower 2)\
B8: Office tower (Office Tower 3)\
B9: Parking lot (Parking Lot)\
BA: Historic office building (Historic Office)\
BB: Corporate headquarters (Corporate Headquarters)

**Industrial 3x3:**\
BC: Chemical processing (Chemical Processing 1)\
BD: Large factory (Large Factory)\
BE: Industrial thingamajig (Industrial Thingamajig)\
BF: Factory (Medium Factory)\
C0: Large warehouse (Large Warehouse 1)\
C1: Warehouse (Large Warehouse 2)

**Misc 3x3:**\
C2: Construction (Construction 1)\
C3: Construction (Construction 2)\
C4: Abandoned building (Abandoned Building 7)\
C5: Abandoned building (Abandoned Building 8)

**Power Plants:**\
C6: Hydro Power (Hydroelectric Power Plant 1)\
C7: Hydro Power (Hydroelectric Power Plant 2)\
C8: Wind Power (Wind Power Plant)\
C9: Gas Power (Gas Power Plant)\
CA: Oil Power (Oil Power Plant)\
CB: Nuclear Power (Nuclear Power Plant)\
CC: Solar Power (Solar Power Plant)\
CD: Microwave Power (Microwave PowerPlant)\
CE: Fusion Power (Fusion Power Plant)\
CF: Coal Power (Coal Power Plant)

**Services:**\
D0: City Hall\
D1: Hospital\
D2: Police Station (Police Dept)\
D3: Fire Station (Fire Dept)\
D4: Museum\
D5: SimPark System (Big Park)\
D6: School\
D7: Stadium\
D8: Prison\
D9: College\
DA: Zoo\
DB: Statue

**Infrastructure (not power plants):**\
DC: Water Pump\
DD: runway (straight)\
DE: Runway intersection\
DF: Pier (Game appears to rotate the pier based on the direction the crane is facing.)\
E0: Crane\
E1: Control Tower (Civilian Control Tower)\
E2: Control Tower (Military Control Tower)\
E3: Warehouse\
E4: Building (Building 1)\
E5: Building (Building 2)\
E6: Tarmac\
E7: F-15b\
E8: Hanger (Hangar 1)\
E9: SimSubway (Subway Station)\
EA: Radar (Tarmac)\
EB: Water Tower\
EC: Bus Station\
ED: SimRail System (Rail Station)\
EE: Parking lot (Civilian Parking Lot)\
EF: Parking lot (Military Parking Lot)\
F0: Loading bay (Loading Bay)\
F1: Top Secret\
F2: Cargo yard (Cargo Yard)\
F3: Mayor’s House\
F4: Water Treatment\
F5: Library System (Library)\
F6: Hangar (Hangar 2)\
F7: Church\
F8: Marina\
F9: Missile silo (Missile Silo)\
FA: Desalinization (Desalinization Plant)

**Arcos:**\
FB: Plymouth Arco (Plymouth Arcology)\
FC: Forest Arco (Forest Arcology)\
FD: Darco (Darco Arcology)\
FE: Launch Arco (Launch Arcology)

**Other:**\
FF: Llama Dome (Braun Llama Dome)

## XZON

1 byte per tile, encodes zone information and also if the tile is the corner of a building.

First 4 bits: encode corners of a building.\
1000: TR\
0100: TL\
0010: BR\
0001: BL

Second 4 bits: encode the following zone information.\
0: none (0000)\
1: Light Residential (0001)\
2: Dense Residential (0010)\
3: Light Commercial (0011)\
4: Dense Commercial (0100)\
5: Light Industrial (0101)\
6: Dense Industrial (0110)\
7: Military (0111)\
8: Airport (1000)\
9: Seaport (1001)\
10 .. 15: Appear to be unused.

_Notes:_\
No city appears to have anything with a hex byte ending in A .. F, so probably unused or saved for future expansion.\
Small park is 1111 0000, no zone but all four corners in one (1x1).

## XUND

Store the underground part of a tile.\
00: None\
01 .. 0F: Subway, indexes as power lines in XBLD.\
10 .. 1E: Pipes, indexes as per power lines in XBLD.\
1F: Crossover, Pipe-TB, Subway-LR\
20: Crossover, Pipe-LR, Subway-TB\
21: unknown, none in 1114 cities.\
22: Missile silo\
23: Sub/rail or Subway Station Underground portion.\
24 .. FF: unknown, none in 1114 cities.

## XTXT

1B per tile. Stores the text overlay for that tile. Note that this overlay is used to store not just signs, but also whether or not a microsim is applied to that tile and neighbour connections.

The "Label?" column indicates if this is an index to the actual text in [XLAB](#xlab) or not.

|Offset|Use|Label?|
|---|---|---|
| 0x00 | No sign | No |
| 0x01 .. 0x32 | 50 user defined sign. | Yes |
| 0x33 | Unused | N/A |
| 0x34 | Bus | Yes |
| 0x35 | Rail | Yes |
| 0x36 | Subway | Yes |
| 0x37 | Wind Power | Yes |
| 0x38 | Hydro Power | Yes |
| 0x39 | Parks | Yes |
| 0x3A | Museum | Yes |
| 0x3B | Library | Yes |
| 0x3C | Marina | Yes |
| 0x3D .. 0xC8 | 139 Microsim labels. See Notes 1 and 2 below. | Yes |
| 0xC9 .. 0xF0 | 39 things treated like signs. Index to [XTHG](#xthg), probably. See Note 3 below. | No |
| 0xF1 .. 0xF9 | Unknown, but seemingly unused. | N/A |
| 0xFA | Neighbour connection | No |
| 0xFB | Toxic Cloud | No |
| 0xFC | Flood | No |
| 0xFD | Rioters | No |
| 0xFE | Rioters (mass riots, maybe?) | No |
| 0xFF | Fire | No |

For `0xFA`, this means that the tile is a neighbour connection, and should have an auto-generated sign on it. These sign values are not user changeable and do not count against the user sign limit of 50.

For `0xFB` and up, these indicate that the tile has the specified disaster tile on it.

**Note 1:** Used for: police, fire, hospitals, schools, stadiums, zoos, prisons, colleges, power plants, water treatment, desalination, mayor’s house, city hall, llama dome, statue, arcos. (Anything that when you click on it, you can change it’s name that isn’t included above.)

**Note 2:** There only appear to be 139 labels here, even if there are 140 user facing microsims. It is unknown what causes this discrepancy.

**Note 3:** Various police/fire/military emergency deploys here, sailboats/nessie, helicopters, maxis man, ships, planes, trains (each train car counts as one). 39 total positions, 33 fire/police/military total. If there are lots of trains and other stuff on the map, this will limit the number of emergency deploys, for example.\
The tornado, monster, crashing airplane as well. Explosions might also appear in here, but may not be saved in the city file.

## XLAB

Labels pointed to by the pointers from [XTXT](#xtxt), except for cases like the disasters, which don't have a text label. 256 total entries.

Labels aren't just signs, but also things like the names for microsims that can have names given to them.

### Label Data Structure

Each entry is a fixed 25 bytes. The first byte is the length of the ASCII data, then up to 23 bytes of ASCII text and finally a null (`/x00`) termination. If the termination is not the 25th byte, the rest is padded out by null data (or, due to a bug in some versions of the game, garbage data).

### Label Indices

Values in quotes are the default in the city file, and they apply to all buildings sharing that microsim. They also apply to all microsims if they are explicitly changed.

|Offset|Label Use|
|---|---|
| 0x00 | Mayor's Name |
| 0x01 .. 0x32 | 50 User Defined Signs |
| 0x33 | Unused label |
| 0x34 | "SimBus System" |
| 0x35 | "SimRail System" |
| 0x36 | "SimSubway" |
| 0x37 | Wind Power |
| 0x38 | Hydro Power |
| 0x39 | "SimPark System" |
| 0x3A | "Museum" |
| 0x3B | "Library System" |
| 0x3C | "Marina" |
| 0x3D .. 0xC8 | 139 [Microsim](#xmic) labels. See Note 2 in [XTXT](#xtxt) section. |
| 0xC9 .. 0xF9 | 48 unused labels |
| 0xFA | Text for the last connection sign, if it exists. |
| 0xFB | Sports team: Football |
| 0xFC | Sports team: Baseball |
| 0xFD | Sports team: Soccer |
| 0xFE | Sports team: Cricket |
| 0xFF | Sports team: Rugby |

Note: Sometimes the unused labels contain garbage data. This can be safely ignored.

## XMIC

150 x 8B microsims\
Probably the first 10 are for the built in microsims, and the next 140 for the rest.

Given there only appear to be 139 microsims pointed to by XLAB, there could be only 139 here as well. More testing is needed to confirm this number.

8B contents:\
00: Index to building type\
01..07: data (varies by building).

    Example: "EC 00 00 18 00 60 0C B3"
    Bus Line, stats 24 (0x0018), 96 (0x0060), 3251 (0x0cb3)
    For a mayor’s mansion: F3 28 08 4A 00 5E 00 00
    F3 = mansion, 28 = 4th stat, 08 4A = 2nd stat, 00=(maybe third stat), 5e = first stat, 00 00 could be padding or employees.
    Llama Dome example: FF 8B 07 C1 01 07 00 80
    8B=first stat, 07C1=2nd, 0107=3rd, 00=?, 80=4th, but 5th in game (# of weddings) is 139 (0x8B)

## XTHG

_Note:_ This section is mostly incomplete and is presently more structured as notes.

00 .. 01: Changes when going from terrain to city mode.

0x0000->0x0402 bit flags? 000000000000->0000010000000010

Doesn’t actually seem to affect the simulation if it’s blanked. However, planes/helicopters seemed to disappear when blanked. Maybe this is a flight path for planes or something?

Last byte was 0x00 in 1114 cities looked at.

Made up of 40x12B chunks.

First chunk appears to be a header

39x other chunks after (39 pointers in XTXT)

Basic structure for a chunk seems to be:

|Offset|Usage|
|---|---|
| 00 | int representing the id of the tile |
| 01 | unknown data, rotation?|
| 02 | unknown data, rotation? (for id=9, this seems to turn the boat into nessie)|
| 03 | tile x coordinate |
| 04 | tile y coordinate |
| 05 | unknown |
| 06 | unknown |
| 07 | unknown |
| 08 | unknown |
| 09 | unknown |
| 0A | unknown |
| 0B | unknown |

**Observed tile ids:**\
0x1: Airplane\
0x2: Helicopter\
0x3: Ship\
0x4: Unknown\
0x5: monster\
0x6: Explosion\
0x7: Police Deploy\
0x8: Fire Deploy\
0x9: Sailboat\
0xA: Train (seems to be front of the train)\
0xB: Train (seems to be for the other two train cars.)\
0xC: Unknown\
0xD: Unknown\
0xE: Military Deploy\
0xF: tornado

## XBIT

1B per tile, storing 8 single bit flags.

|Bit Position | Flag Meaning|
|---|---|
| 0 | Powerable (Does this tile receive power).|
| 1 | Powered (Is this tile receiving power).|
| 2 | Piped (Can this tile receive water).|
| 3 | Watered (Is this tile receiving water).|
| 4 | XVAL mask. |
| 5 | Water (is this tile covered in water).|
| 6 | Rotate the tile by 90 degrees.|
| 7 | Salt Water (Will water on this tile be fresh or salt water).|

_Bit #4 Notes:_  This bit is set for a mask for xval. 64x64 and needs scaling. Doesn’t appear in all cities. Ignoring it does not seem to cause issues as value 0x00 in xval is already transparent. Maybe only old/DOS created cities.\
_Rotation Notes:_ For example, LR pier becomes TB peir with this set, LR runway becomes TB with this set, LR onramps become TB onramps with this set, bridge pieces as well.

## 64x64  Minimaps

Total # is 4096 = 64x64.

Show in city appears to split into 2x2 blocks, for a total of 4096 blocks.\
Going with 16 colours from white to black makes bitmaps that are very similar to the in game minimap.

### XTRF

Traffic data. This is why the game shows traffic in 2x2 blocks, even for roads that are only 1x1 tiles in size.

### XPLT

Pollution

### XVAL

Land Value, simulation shows these values using human friendly 1 indexed.

### XCRM

Crime

## 32x32 Minimaps
Total # is 1024 = 32x32. Show in city appears to split into 4x4 blocks, for a total of 1024 blocks.\
Minimap window seems to bin these in 16 even blocks.

### XPLC

Police coverage

### XFIR

Fire coverage.

### XPOP

Population density.

### XROG

Rate of Growth graph information.

7F: no change (7F seems to be the default for a new city what hasn’t been run yet).|
>82: + change (Green)|
<7D: - change (Red)

_Note:_ Numbers chosen after experimentation, numbers in the very middle could affect the simulation, no 00 or FF noted.

### XGRP

Historical graph data for the graph window.\
16 different tracked stats, shows 1, 10 and 100 years. Appears to update every month, every 1/2 year and every 5 years.\
16*(20+20+12)=832*4B=3328B

**Contents:**\
Each section contains 52*4B integers containing historical data for the graph.\
12 for 1 year, 20 for 10 years, 20 for 100 years. (1/month, 1/6months, 1/5years)

From beginning:

|Offset|Graph|
|---|---|
|0000 .. 00CF | City Size|
|00D0 .. 019F | Residents|
|01A0 .. 026F | Commerce|
|0270 .. 033F | Industry|
|0340 .. 040F | Traffic.|
|0410 .. 04DF | Pollution.|
|04E0 .. 05AF | Value|
|05B0 .. 067F | Crime|
|0680 .. 074F | Power %|
|0750 .. 081F | Water %|
|0820 .. 08EF | Health|
|08F0 .. 09BF | Education|
|09C0 .. 0A8F | Unemployment|
|0A90 .. 0B5F | GNF (GNP?), in 1000s.|
|0B60 .. 0C2F | Nat’n Pop., in 1000s|
|0C30 .. 0CFF | Fed Rate|

## Additional Fields for Scenario Files

### TEXT

Textual description of the scenario. Max lengths are unknown.

There are two entries. The first four bytes describe the type of entry:

- `80 00 00 00`: the scenario selection screen description.
- `81 00 00 00`: the extended description shown in game when the scenario starts.

### SCEN

Scenario information. This includes which disaster to trigger and where, as well as win conditions and the time limit

The entries after the time limit are the goals that need to be met before that time limit expires.

| Offset | Length | Type | Use | Notes |
|---|---|---|---|---|
| 0x00 | 4B | Bytes | Header | Header, always `80 00 00 00`. |
| 0x04 | 2B | Integer | Disaster | Type of disaster, as defined in the MISC section. |
| 0x06 | 1B | Integer | Disaster X Location |  |
| 0x07 | 1B | Integer | Disaster Y Location |  |
| 0x08 | 2B | Integer | Time Limit (Months) |  |
| 0x0A | 4B | Integer | City Size | Seems to be comparing against the normal city population without arcos from MISC. |
| 0x0E | 4B | Integer | Residential Population |  |
| 0x12 | 4B | Integer | Commercial Population  |  |
| 0x16 | 4B | Integer | Industrial Population |  |
| 0x1A | 4B | Integer | Cash Goal | Seems to be total cash on hand, not counting debts from bonds. |
| 0x1E | 4B | Integer | Land Value Goal | Total city land value should be above this. |
| 0x22 | 4B | Integer | Pollution Limit | Pollution should be below this limit. |
| 0x26 | 4B | Integer | Crime Limit | Crime should be below this limit. |
| 0x2A | 4B | Integer | Traffic Limit | Traffic should be below this limit. |
| 0x2E | 1B | Integer | First Building | Building ID per [XBLD](#xbld) to be built to win. |
| 0x2F | 1B | Integer | Second Building | As above, but a second goal. |
| 0x30 | 2B | Integer | First Building Tile Count | Count of tiles that need to be built to win. Untested, but this means that building a single 4x4 building would require 16 tiles to be built to win. |
| 0x32 | 2B | Integer | Second Building Tile Count | As above, but for second building goal. |
| 0x34 | 4B | Null | Padding (Optional) | Sometimes there’s an extra 4 bytes of 0s at the end of the scenario as padding, but these don't seem appear in all scenarios. Potentially one version of the game needed a specific alignment. |

### PICT

Image data for the scenario. All of the observed scenarios appear to be 65x65 pictures, with a 1px border on them, making the effective size actually 63x63.

**Data Format:**

| Offset | Length | Use |
|---|---|---|
| 0x00 | 4B | Header, always `80 00 00 00` |
| 0x04 | 2B | X size of image in pixels. |
| 0x06 | 2B | Y size of image in pixels. |
| 0x08 | varies | Rows of image data. |

Each row of image data is the Y dimension single pixel bytes, with an additional `0xFF` denoting the end of a row. Additionally, the first and last row being all `0x01` and the first and last pixel of all other rows being `0x01`.

The pixel bytes are colours chosen from an internal palette, seemingly based on `PAL_MSTR.BMP` as used for all other ingame graphics. There seems to be a -16 offset from the indices used to render the tile sprites.

Not all values in the `PAL_MSTR.BMP` are actually valid values, with the observed range being from 18 to 169 inclusive. Additionally, values in the range 160 to 170 (in `PAL_MSTR`) appear to be remapped to indices 0 to 10. Note that 170 is where the first colour cycling value appears in `PAL_MSTR.BMP`.

There are also two special case remapping between the `PAL_MSTR.BMP` and the PICT palette index:

- (0, 0, 0): 0
- (127, 127, 127): 254

Note: The game does not appear to actually care if the "border" values are `0x01` as `0x00` has been observed to work as well. It also doesn't appear to require the `0xFF` byte at the end of a row of pixels, instead seeming to use the X/Y dimensions specified in the header to determine pixel data. Only older scenarios seem to use the explicit border and row end values, with newer ones using all `0x00`.

Note 2: Some images appear to have full black (0, 0, 0) in them, but enumeration of the palette did not produce the black pixels, and it is presently unclear how this works.

This image shows the colours produced for each value between 0 and 255 (inclusive). For any value without a colour mapping, the game renders it as either (127, 127, 17) or (159, 159, 159). It is unclear why or how it chooses differently between the two. As noted previously 254 seems to be the "real" index for the mid-gray, but it's unclear which is the "real" for the 159 gray.

![PICT image index to palette mapping.](images/pict_palette.png)

### TMPL

This only appears in the 5 scenarios originally shipped with the game.

Each entry consists of a pascal style string, followed by a data type in a 4-character shortened format.

It has no function in actual game usage, but lays out what the contents of the `SCEN` section contain, presumably so that people can make their own scenarios. It is unknown why later scenarios omit this information. This information is identical to the [SCEN](#scen) section of this documentation.
