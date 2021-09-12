# Text File Specifications

Covers the contents of the newspapers in DATA_USA and [other ingame text](#other-ingame-text) in TEXT_USA (though it should work for other localizations, only tested with the USA version). Written for the Windows 95 Special Edition version, with differences noted. The DOS version of the data files is different, but the text and rules inside appear to be the same, barring some minor differences.

## Newspapers

DATA_USA.DAT has the raw newspaper text, and some metadata on how to build stories.
DATA_USA.IDX has an index for reading the various parts of the newspaper data file.
This specification is incomplete in exactly how this file works, but is enough to figure out what is going on.

### Basic format specification:

DATA_USA.IDX contains offsets for certain sections in the DATA_USA.DAT file. This is used to break segments into specific lengths. The names for each section come from the Mac version of SimCity 2000, where this data is stored in DATA resources in the resource fork.

| id    | length | name          | purpose                      |
|-------|--------|---------------|------------------------------|
| 0x3e8 | 500    | Group Start   | start points of token groups |
| 0x3e9 | 500    | Group Count   | token counts in each group   |
| 0x3ea | 10000  | Token Pointer | pointers to each token       |
| 0x3eb | 149227 | Token Data    | actual newspaper data        |
| 0x3ec | 160    | Story Power   | unknown                      |
| 0x3ed | 160    | Story Decay   | unknown                      |

#### Token Pointer

A series of 32-bit values, stored big-endian. Each value is an offset into the `Token Data` section, and defines the start of a token.

#### Token Data

The text data itself. Each individual token is terminated with `0x00`.

#### Group Start and Group Count

These segments contain a series of two-byte values, stored big-endian. Each value in `Group Start` stores the index of a token from the `Token Pointer` section, and indicates the start of a group. The corresponding value in the `Group Count` segment contains the number of tokens in each group.

#### Story Power and Decay

Unknown at present. There are many repeated values that are about right to be 8B integers. More work needed to figure out what these do.

### Compression

Uses a simple substitution compression to substitute two letters next to each other (and taking two bytes) with a single byte token. Potentially chosen based on analysis of the input data. Some values are also meant to be substituted with other things, such as they Mayor's name or city name.

#### Byte -> Value Mapping Table:

Note that all-caps values are not something to decompress, but a pointer to fill in a word from a different spot in the data file.

| Value | Output    | Value | Output     | Value | Output | Value | Output |
|-------|-----------|-------|------------|-------|--------|-------|--------|
| 0x0   | None      | 0x40  | @          | 0x80  |        | 0xC0  | di     |
| 0x1   | th        | 0x41  | A          | 0x81  |        | 0xC1  | mo     |
| 0x2   | in        | 0x42  | B          | 0x82  |        | 0xC2  | bu     |
| 0x3   | re        | 0x43  | C          | 0x83  |        | 0xC3  | ho     |
| 0x4   | to        | 0x44  | D          | 0x84  |        | 0xC4  | ea     |
| 0x5   | an        | 0x45  | E          | 0x85  |        | 0xC5  | un     |
| 0x6   | at        | 0x46  | F          | 0x86  |        | 0xC6  |        |
| 0x7   | er        | 0x47  | G          | 0x87  |        | 0xC7  |        |
| 0x8   | st        | 0x48  | H          | 0x88  |        | 0xC8  | io     |
| 0x9   | en        | 0x49  | I          | 0x89  |        | 0xC9  | pa     |
| 0xA   | on        | 0x4A  | J          | 0x8A  |        | 0xCA  | ra     |
| 0xB   | of        | 0x4B  | K          | 0x8B  |        | 0xCB  | ke     |
| 0xC   | te        | 0x4C  | L          | 0x8C  |        | 0xCC  | lo     |
| 0xD   | ed        | 0x4D  | M          | 0x8D  |        | 0xCD  | ly     |
| 0xE   | ar        | 0x4E  | N          | 0x8E  |        | 0xCE  | ri     |
| 0xF   | is        | 0x4F  | O          | 0x8F  |        | 0xCF  | sh     |
| 0x10  | ng        | 0x50  | P          | 0x90  |        | 0xD0  |        |
| 0x11  | me        | 0x51  | Q          | 0x91  |        | 0xD1  |        |
| 0x12  | co        | 0x52  | R          | 0x92  |        | 0xD2  |        |
| 0x13  | ou        | 0x53  | S          | 0x93  |        | 0xD3  |        |
| 0x14  | al        | 0x54  | T          | 0x94  |        | 0xD4  |        |
| 0x15  | ti        | 0x55  | U          | 0x95  |        | 0xD5  |        |
| 0x16  | es        | 0x56  | V          | 0x96  |        | 0xD6  |        |
| 0x17  | ll        | 0x57  | W          | 0x97  |        | 0xD7  |        |
| 0x18  | he        | 0x58  | X          | 0x98  |        | 0xD8  |        |
| 0x19  | ha        | 0x59  | Y          | 0x99  |        | 0xD9  | pe     |
| 0x1A  | it        | 0x5A  | Z          | 0x9A  |        | 0xDA  | ch     |
| 0x1B  | ca        | 0x5B  | [          | 0x9B  |        | 0xDB  | tr     |
| 0x1C  | ve        | 0x5C  | \          | 0x9C  |        | 0xDC  | ci     |
| 0x1D  | fo        | 0x5D  | ]          | 0x9D  |        | 0xDD  | hi     |
| 0x1E  | de        | 0x5E  | ^          | 0x9E  | le     | 0xDE  |        |
| 0x1F  | be        | 0x5F  | _          | 0x9F  | or     | 0xDF  | so     |
| 0x20  | `space`   | 0x60  | \`         | 0xA0  |        | 0xE0  |        |
| 0x21  | !         | 0x61  | a          | 0xA1  |        | 0xE1  |        |
| 0x22  | "         | 0x62  | b          | 0xA2  |        | 0xE2  |        |
| 0x23  | #         | 0x63  | c          | 0xA3  |        | 0xE3  |        |
| 0x24  | $         | 0x64  | d          | 0xA4  |        | 0xE4  |        |
| 0x25  | %         | 0x65  | e          | 0xA5  |        | 0xE5  |        |
| 0x26  | &         | 0x66  | f          | 0xA6  |        | 0xE6  |        |
| 0x27  | '         | 0x67  | g          | 0xA7  |        | 0xE7  |        |
| 0x28  | (         | 0x68  | h          | 0xA8  |        | 0xE8  |        |
| 0x29  | )         | 0x69  | i          | 0xA9  | ma     | 0xE9  |        |
| 0x2A  | *         | 0x6A  | j          | 0xAA  | li     | 0xEA  |        |
| 0x2B  | +         | 0x6B  | k          | 0xAB  | we     | 0xEB  |        |
| 0x2C  | ,         | 0x6C  | l          | 0xAC  | ne     | 0xEC  |        |
| 0x2D  | -         | 0x6D  | m          | 0xAD  |        | 0xED  |        |
| 0x2E  | .         | 0x6E  | n          | 0xAE  | se     | 0xEE  | su     |
| 0x2F  | /         | 0x6F  | o          | 0xAF  | nt     | 0xEF  | rt     |
| 0x30  | 0         | 0x70  | p          | 0xB0  | wa     | 0xF0  | ta     |
| 0x31  | 1         | 0x71  | q          | 0xB1  | wh     | 0xF1  | ge     |
| 0x32  | 2         | 0x72  | r          | 0xB2  | pr     | 0xF2  | rs     |
| 0x33  | 3         | 0x73  | s          | 0xB3  | si     | 0xF3  | ow     |
| 0x34  | 4         | 0x74  | t          | 0xB4  | as     | 0xF4  | us     |
| 0x35  | 5         | 0x75  | u          | 0xB5  |        | 0xF5  | ss     |
| 0x36  | 6         | 0x76  | v          | 0xB6  |        | 0xF6  | sp     |
| 0x37  | 7         | 0x77  | w          | 0xB7  |        | 0xF7  | ac     |
| 0x38  | 8         | 0x78  | x          | 0xB8  | nd     | 0xF8  | il     |
| 0x39  | 9         | 0x79  | y          | 0xB9  | po     | 0xF9  | ic     |
| 0x3A  | :         | 0x7A  | z          | 0xBA  | la     | 0xFA  | pl     |
| 0x3B  | ;         | 0x7B  | {          | 0xBB  | no     | 0xFB  | fe     |
| 0x3C  | <         | 0x7C  |            | 0xBC  | ce     | 0xFC  | wo     |
| 0x3D  | CITYNAME  | 0x7D  | }          | 0xBD  | fi     | 0xFD  | da     |
| 0x3E  | TEAMNAME  | 0x7E  | MAYORNAME  | 0xBE  | yo     | 0xFE  | ai     |
| 0x3F  | ?         | 0x7F  | wi         | 0xBF  | do     | 0xFF  | ur     |

Notes:

- `0x20`: is " ", the space character,
- `0x3E`: team name is probably one of the sports team names, unclear how this is chosen from the available ones.

### Other mappings

#### Escape Characters

`[` byte is an escape value. This means instead of using the previous lookup table, instead treat it as a pointer to some other piece of data in the [escape values table](#escape-value-table) below.

`^` seems to behave similarly. `^0x97` is a country name just like `[0x97`, but it's used to disambiguate between two different countries that should stay the same within the news story.

`@` also `@0xa4` seems to be a foreign last name. Just link `[0xa4`.

`*` seems to behave slightly differently. With the previous escape characters, when a value is selected the same value will be used throughout the article. For instance, every instance of `[0xB0` will map to the same last name. However, `*0xB0` will pick a random last name for each occurrence in the article. If the `*` is in a headline, the occurences in the body should match. This is not the case in the DOS version.

#### Numbers

`%x` is some number. X seems to be the maximum the number will be. Multiple numbers can be stacked together, for example in the time templates: `%12:%5%9 am` and `%12:%5%9 pm`. The number can contain a 1000s seperator, like `%250,000`. The minimum appears to be 1.

There appears to be a bug in the Windows 95 version affecting number generation. The Mac and DOS versions correctly generate number in the right range. This may be due to running the game on modern Windows versions, perhaps the random function changed.

Broken examples from the Win95 version.: %100=1200, %1000 = 7000, %20 = 30,370, %50 = 360, %250 = 1350, %4 = 30 %250,000 = 2350,000

`<x` seems to behave the same way, but seems to only be used for death tolls for disasters.

`$%` (without any numbers after it) seems to only occur in the headline for new articles reporting that the federal interest rate has been raised or lowered, and is filled in from the game data.

`x%` is a literal percentage, and is not replaced.

#### Group Indexes Table

This is for storing general news stories. Any token replacements are handled normally. Values `0x4B` and up are token escape values and are in the next section. 66 total.

Some of the stores are just general stories, such as those between `0x01` and `0x0F` intended to be filler stories, except the two sets dealing with inventions, which announce that a new piece of technology is available.

There are some general/filler stories that aren't directly related to the game:

- `0x01` to `0x0F`: general stories, except `0x03` and `0x04`.
- `0x2D` are questions for the Miss Sim advice column.

Many stories are generated based on game conditions and events:

- `gen`: is token replacement to generate a dynamic story, and contain no story text themselves. These are based on game events.
- `disaster`: are those shown after a disaster.
- `fund`: if funding is too low for that area.
- `need`: if more of a service is needed. Recreation needs are treated interchangeably by the game, but have unique stories.
- `invention`: when a technology is invented and available in the game.
- `low`/`high`: when a threshold is crossed, either high or low about one of the major simulation variables.
  - `0x10` to `0x15` are negative stories.
  - `0x3D` to `0x42` are positive stories.
- `0x25`: warning that a power plant is getting old.
- `0x28`: citizen's don't like tree removal.
- `0x3D` to `0x42`: are positive stories about the respective area.

All of these, that aren't just content-less pointer tokens, contain at minimum a headline and some content, or pointers to both.

| Value | Group                | Value | Group                | Value | Group                | Value | Group                |
|-------|----------------------|-------|----------------------|-------|----------------------|-------|----------------------|
| 0x00  | genweatherstory      | 0x11  | hightraffic          | 0x22  | disasterhurricane    | 0x33  | needhospital         |
| 0x01  | sciencestory         | 0x12  | highpollution        | 0x23  | disasterriot         | 0x34  | needschool           |
| 0x02  | foundingstory        | 0x13  | loweducation         | 0x24  | oldpowerplant        | 0x35  | needseaport          |
| 0x03  | genpopulationstory   | 0x14  | lowhealthcare        | 0x25  | fundprison           | 0x36  | needairport          |
| 0x04  | invention2story      | 0x15  | highunemployment     | 0x26  | fundeducation        | 0x37  | needzoo              |
| 0x05  | inventionstory       | 0x16  | disasterfire         | 0x27  | fundtransit          | 0x38  | needstadium          |
| 0x06  | warstory             | 0x17  | disasterflood        | 0x28  | treeremoval          | 0x39  | needmarina           |
| 0x07  | businessstory        | 0x18  | disasterplane        | 0x29  | gennewordinancestory | 0x3A  | needpark             |
| 0x08  | sportsstory          | 0x19  | disasterhelicopter   | 0x2A  | genrichopoinionstory | 0x3B  | needseapoer          |
| 0x09  | fedrateupstory       | 0x1A  | disastertornado      | 0x2B  | genrichgripestory    | 0x3C  | needconnections      |
| 0x0A  | fedratedownstory     | 0x1B  | disastereqarthquake  | 0x2C  | genopiniongripe      | 0x3D  | lowcrime             |
| 0x0B  | nationalpolitical    | 0x1C  | disastermonster      | 0x2D  | missimquestion       | 0x3E  | highrating           |
| 0x0C  | internationalstory   | 0x1D  | disastermeltdown     | 0x2E  | needpower            | 0x3F  | lowpollution         |
| 0x0D  | gendisasterstory     | 0x1E  | disastermicrowave    | 0x2F  | needroad             | 0x40  | higheducation        |
| 0x0E  | healthstory          | 0x1F  | disastervolcano      | 0x30  | needpolice           | 0x41  | highhealth           |
| 0x0F  | localpoliticalstory  | 0x20  | disasterspill        | 0x31  | needfire             | 0x42  | lowunemployment      |
| 0x10  | highcrime            | 0x21  | disastermajorspill   | 0x32  | needwater            |       |                      |

#### Escape Value Table

The values here refer to the index of a group from the `Group Start` segment. To find a replacement, simply pick a token from the referenced group. 40 total.

#### Complex Tokens

These tokens are still shorter than a full news story, but they're more complex than the simple tokens. They tend to contain either opinions on a topic (such as `0x5C` to `0x62`), or other general headlines.

`0x43` and `0x44` based on new ordinances passed.

Headlines for dynamically generated stories are stored in `0x64` onwards.

Group `0x63` is a special case and contains token replacements point back to other news stories dealing with disasters in **other** cities. This is only used in group `0x0D` from the previous section.

| Value | Group              | Value | Group              | Value | Group              | Value | Group              |
|-------|--------------------|-------|--------------------|-------|--------------------|-------|--------------------|
| 0x43  | ordinance          | 0x4D  | reaction           | 0x57  | gripe3             | 0x61  | education          |
| 0x44  | ordinanceopinion   | 0x4E  | missimanswer       | 0x58  | gripe4             | 0x62  | health             |
| 0x45  | otherfire          | 0x4F  | gripeclosing       | 0x59  | gripe5             | 0x63  | articlepointers    |
| 0x46  | otherflood         | 0x50  | weatherreport      | 0x5A  | gripe6             | 0x64  | ordinanceheadline  |
| 0x47  | otherrailcrash     | 0x51  | weatherprediction  | 0x5B  | gripe7             | 0x65  | weatherheadline    |
| 0x48  | othertornado       | 0x52  | richopinion        | 0x5C  | traffic            | 0x66  | populationheadline |
| 0x49  | otherearthquake    | 0x53  | richgripe          | 0x5D  | pollution          | 0x67  | gripeheadline1     |
| 0x4A  | othermonster       | 0x54  | opiniongripe       | 0x5E  | crime              | 0x68  | gripeheadline2     |
| 0x4B  | foundingreaction   | 0x55  | gripe1             | 0x5F  | taxes              | 0x69  | gripeheadline3     |
| 0x4C  | populationstory    | 0x56  | gripe2             | 0x60  | employment         | 0x6A  | disasterheadline   |

#### Simple Tokens

These tokens are either single words or short phrases to fill in spots in a news story. 100 total.

| Value | Group             | Value | Group             | Value | Group             | Value | Group             |
|-------|-------------------|-------|-------------------|-------|-------------------|-------|-------------------|
| 0x6B  | destroyverb       | 0x84  | adjective         | 0x9D  | exclamation       | 0xB6  | moneything        |
| 0x6C  | creationverb      | 0x85  | adverb1           | 0x9E  | expert            | 0xB7  | moneyevent        |
| 0x6D  | pastcreationverb  | 0x86  | adverb2           | 0x9F  | emotion           | 0xB8  | money             |
| 0x6E  | fireverb          | 0x87  | time              | 0xA0  | compete           | 0xB9  | month             |
| 0x6F  | increase          | 0x88  | negativefeeling   | 0xA1  | malename          | 0xBA  | positive          |
| 0x70  | verb1             | 0x89  | animal            | 0xA2  | femalename        | 0xBB  | numberword        |
| 0x71  | verb2             | 0x8A  | negativeadjective | 0xA3  | foreignfirstname  | 0xBC  | noun              |
| 0x72  | futureverb        | 0x8B  | bodypart          | 0xA4  | foreignlastname   | 0xBD  | job               |
| 0x73  | makeverb          | 0x8C  | disaster          | 0xA5  | 2xforeignname     | 0xBE  | territory         |
| 0x74  | pastmakeverb      | 0x8D  | bignumber         | 0xA6  | name              | 0xBF  | powertype         |
| 0x75  | makenoun          | 0x8E  | locals            | 0xA7  | ngo               | 0xC0  | relative          |
| 0x76  | response          | 0x8F  | localgov          | 0xA8  | room              | 0xC1  | smalladjective    |
| 0x77  | utterance         | 0x90  | landmark          | 0xA9  | numericposition   | 0xC2  | sport             |
| 0x78  | feeling           | 0x91  | othercity         | 0xAA  | invention         | 0xC3  | injury            |
| 0x79  | scareverb         | 0x92  | localcity         | 0xAB  | invention2        | 0xC4  | sportscore        |
| 0x7A  | violentverb       | 0x93  | response3         | 0xAC  | politicalissue    | 0xC5  | teamname          |
| 0x7B  | pastviolentverb   | 0x94  | shortsaying       | 0xAD  | product           | 0xC6  | business          |
| 0x7C  | observedverb      | 0x95  | action1           | 0xAE  | size              | 0xC7  | location          |
| 0x7D  | angerverb         | 0x96  | action2           | 0xAF  | title             | 0xC8  | research          |
| 0x7E  | presentverb1      | 0x97  | country           | 0xB0  | lname             | 0xC9  | badthing          |
| 0x7F  | want              | 0x98  | criminal          | 0xB1  | legalthing        | 0xCA  | waterbody         |
| 0x80  | pastwant          | 0x99  | crimetype         | 0xB2  | llama             | 0xCB  | group             |
| 0x81  | negativeverb      | 0x9A  | infrastructure    | 0xB3  | illness           | 0xCC  | governmentthing   |
| 0x82  | pastverb          | 0x9B  | direction         | 0xB4  | amount            | 0xCD  | weathercondition  |
| 0x83  | thingdescriptor   | 0x9C  | illness2          | 0xB5  | importantperson   |       |                   |

### News Story Formatting

Before the game displays the text, it is formatted according to a few simple rules (possibly incomplete):

- Any text that comes before the first plus `+` sign is the headline for that news story, and text is transformed into title case with a space and a dash at the end. For example: `llama attacks on the rise` becomes `Llama Attacks On The Rise -`.
- An single `-` indicates the start of a paragraph. This results in the game displaying a newline, with the following text indented. The next letter should be uppercase.
  - `-` in between two words is not treated as a paragraph break.
- The first letter after a `.` should be uppercase. This means that the fragment `. very good.` becomes `. Very good.`
- `--` is is either rendered as a `--` or two paragraph breaks in the game. If it's in a MisSim question, it's rendered as two breaks. If it's in the body of text otherwise, it's rendered as dashes.

### How Newspaper Stories Are Created

This is incomplete. It is unclear how multiple tokens are handled. It's probably random.

1. Some event triggers the news story to be generated. For filler stories, this is probably just part of normal newpaper generator. This created an index to look at a token group.
2. Within the token group, one or more tokens are present. If there is only one, choose it. If there are multiple, choose one.
3. With the chosen token, replace any replacement values with a token using this same algorithm.
4. Format the text.

For step 3, the replacements should all be in the token escape tables or covered in [numbers](#numbers).

## Other Ingame Text

Stored in TEXT_USA.DAT with an index in TEXT_USA.IDX

Format is relatively straightforward:\
The index file contains an ID for the text (likely to identify it through some in-game mechanism) and a length of the ASCII text block.\
Parsing the data file just involves splitting it up into chunks with the given lengths and converting the RAW bytes to ASCII characters. Note that there seems to be some characters outside the ASCII range, ignoring them doesn't seem to cause any ill effects in parsing.
