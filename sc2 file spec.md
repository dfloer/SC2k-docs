# SimCity 2000 Windows 95 .sc2 File Specification

## Basic File Layout

The basic file structure is of an EA IFF file: \
It starts with a 12B (byte) header which contains:
- First 4B is the IFF type, for .sc2 files this is FORM.
- Next 4B is the length of the file (not counting the first 12B)
- Last 4B is a container for the rest of the file, for .sc2 files this is SCDH.

## Chunks:

Rest of the file has a 4B chunk id (which is 4 ASCII characters between 0x20 and 0x7F inclusive), a 4B integer size for the chunk and finally the chunk data of the same length, in bytes, as the size of the compressed chunk.

#### Chunks in the file and their uncompressed lengths:

- [CNAM](#cnam): 32 (Stored uncompressed. May be optional in Mac files.)
- [MISC](#misc): 4800    
- ALTM: 32768 (Stored uncompressed.)
- XTER: 16384    
- XBLD: 16384    
- XZON: 16384
- XUND: 16384
- XTXT: 16384
- XLAB: 6400
- XMIC: 1200
- XTHG: 480
- XBIT: 16384
- XTRF: 4096
- XPLT: 4096
- XVAL: 4096
- XCRM: 4096
- XPLC: 1024
- XFIR: 1024
- XPOP: 1024
- XROG: 1024
- XGRP: 3328

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
|0008 | Rotation | Internally called compass. Number between 0 and 3 corresponding to number of counterclockwise rotations.|
|000C | Year Founded | Year city was founded.|
|0010 | City Age | days since city was founded in 300 day years, and 25 day months? example: date=2435, founded= 2050, (2435-2050)=385. month=july=07, so 385(300)+7*25=115,650. In save file: 115,663, so probably a few day into July.|
|0014 | Money | Stored as signed 32b int |
|0018 | Number of bonds. |
|001C |  Game Level | This seems related to the difficulty the game was started with.|
|0020 |  City Status | Unknown |
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
|005C| Null? | Possibly Null |
|0060| Heat | Something weather related.|
|0064 | Wind | Something weather related. |
|0068 | Humid | Something weather related. |
|006C | Weather | Called weatherTrend by the game. See [Weather Type table](#weather-type).|
|0070 | Disasters | See [Disaster Type table](#disaster-type).|
|0074 | Residential population| Unknown exactly.|
|0078 | Rewards availability | (0s)11111= all 5, 00001=mayor’s only, 10000=arcos only, etc.|
|007C .. 0168 | Population/Health/Education Graph Data| 20 values each, interleaved in that order.|
|016C .. 01EC | Industry Graph Data | 33x4B. Each appears to be for the industry window. There are 3 different graphs, each of which appears to show be stored as ratios, tax rate, demand. _Ranges on values unknown._|
|01F0 .. 05EC | Building Counts | Same index as XBLD, count of that # of building in the city.|
|05F0 | Populated Tile Count | Total number of populated tiles.|
|05F4 | ? | Unknown, but seems like it'd be the other part of Populated Tile count |
|05F8 | Residential Tile Count | |
|05FC | ? | Something like it should be the other half of Residential Tile Count.|
|0600 | Commercial Tile Count | |
|0604 | ? | Something like the other half of commerial tile count.|
|0608 | Industrial Tile Count | |
|060C | ? | Something like the other half of industrial tile count.|
|0610 .. 06D7 | Bond Data | bonds, maximum of 50, signed 32b int. Famous overflow in game. |
|06D8 .. 0718 | Neighour Data | 4x4B of neighbour information. Form is: neighbour index, neighbour population, neighbour value (unknown what exactly this is) and neighbour fame (again unknown). Ordering is: lower left, upper left,  unknown,  upper right,  bottom right.|
|0710 | Unknown | Seems to be 0 in an established city (full RCI), +’ve in others. Unknown exactly.|
|0718 .. 0720 | RCI demand | Signed 32b int from -1999 to +2000. First R, second C, third I.|
|0738 .. 0778 | Technology Discovery Years | Contains the year the technology was discovered. Appears to be 0 if the city was saved after the technology was invented. Details in [Technology Discovery Years Table](#technology discovery years)|
|077C .. 08BF | Property taxes | Each takes 27*4B. See [Tax Rate Table](#tax rate details) |
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
| 0E4C | Military Base | Not offered base: 0, offered base but no suitable location: 1, 2: army base, 3: airbase, 4 navy: base, 5: missile silos |
| 0E50 .. 0EC4 |Newspaper List | Unknown exactly, 6x5B structure.|
| 0EC8 .. 0EFC | Newspaper List | Unknown exactly, 9x6B structure.|
| 0FA0 | Ordinances flags | Bit flags for which of the 20 ordinances are enacted. 00000000:none, 000fffff:20. First ordinances (finance) section are rightmost bits. |
| 0FA4 | unemployed | Unknown |
| 0FA8 .. 0FE4 | Military Count | Unsure, but it may be a 16x2B count of military tiles.
| 0FE8 | Subway Count | Unknown |
| 0FEC | Speed | Speed setting: 1=paused, 2=Turtle, 3=Llama, 4=Cheetah, 5=African Swallow |
| 0FF0 | Auto Budget | Auto budget setting. |
| 0FF4 | Auto Goto | Auto goto setting. |
| 0FF8 | Sound | Sound effects setting. |
| 0FFC | Music | Music setting. |
| 1000 | Disasters | No disasters setting. |
| 1004 | Paper Delivery | Is paper delivery enabled. |
| 1008 | Extra Newaspaper | Is the Extra!!! newspaper enabled. |
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
| 1038 | Old Arrest | Unknown |
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
|0764 | Desalinisation | |
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
		
#### Budget City Service Details

Each segment has 27 x 4B entries structered.
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

