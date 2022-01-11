# Simulations Specifications

## About these specifications:

At this point, these specifications are largely structured as notes rather an implementable specification due to the very incomplete nature of them. Written for the Windows 95 Special Edition version.

### Starting a New City:

The simulation is initialized with several values:
- The start date to national population mapping is:
  - 1900 : 10,000.
  - 1950 : 25,000.
  - 2000 : 60,000.
  - 2050 : 150,000.
- National value is: 3 * national population / 10.
- National tax rate always starts at 1.
- The national trend (whose effects are currently unknown) is the difficulty value - 1.
- Difficulty:
  1. starting funds: $20,000.
  2. starting funds: $10,000.
  3. starting funds: $10,000. This is a bond at 3% interest.
- Invention dates are calculated from the base year they can be [invented](https://github.com/dfloer/SC2k-docs/blob/master/sc2%20file%20spec.md#technology-discovery-years) with an additional 0 to 19 years randomly added.

### Terrain Generation:

#### Terrain Generation settings:

| Name | Range | Default |
| --- | --- | --- |
| Coast | 0 or 1 | 0 |
| River | 0 or 1 | 0 |
| Hills | 0 to 47 | 12 |
| Water | 0 to 47 | 5 |
| Trees | 0 to 47 | 15 |

Selecting Coast also allow salt-water to be placed on the map.

### Power Plants:

Methodology to determine: determined by building a city with low density zones and calculating how many tiles each power plant could power. Note that these numbers include the power plant, as it seems to require power to operate.

|Plant|Nominal Output (MW)|Actual Output (tiles)|Efficiency|
|---|---|---|---|
| Coal | 200 | 704 | 44 |
| Hydroelectric | 20 | 40 | 40 |
| Oil | 220 | 768 | 48 |
| Gas | 50 | 176 | 11 |
| Nuclear | 500 | 1776 | 111 |
| Microwave | 1600 | 5680 | 355 |
| Fusion | 2500 | 8880 | 555 |

Efficiency is simply the number of tiles a plant can power / the number of tiles the plant takes up. So for the Coal plant this is 704 / 16 = 44. This is not part of the simulation, just included for illustrative purposes.

##### Wind Power:

Methodology to determine: A city was made with all 32 terrain levels to determine how elevation affects output.

The power output of a wind turbine is: `altitude // 2 + [0, 3]` where the 0-3 seems to be partially determined by how windy it is, normalized to a 0-32 range with some randomness added.

| Altitude (steps) |Altitude (feet) | Minimum Tiles Powered | Maximum Tiles Powered |
|---|---|---|---|
| 0 | 50 | 0 | 3 |
| 1 | 150 | 0 | 3 |
| 2 | 250 | 1 | 4 |
| 3 | 350 | 1 | 4 |
| 4 | 450 | 2 | 5 |
| 5 | 550 | 2 | 5 |
| 6 | 650 | 3 | 6 |
| 7 | 750 | 3 | 6 |
| 8 | 850 | 4 | 7 |
| 9 | 950 | 4 | 7 |
| 10 | 1050 | 5 | 8 |
| 11 | 1150 | 5 | 8 |
| 12 | 1250 | 6 | 9 |
| 13 | 1350 | 6 | 9 |
| 14 | 1450 | 7 | 10 |
| 15 | 1550 | 7 | 10 |
| 16 | 1650 | 8 | 11 |
| 17 | 1750 | 8 | 11 |
| 18 | 1850 | 9 | 12 |
| 19 | 1950 | 9 | 12 |
| 20 | 2050 | 10 | 13 |
| 21 | 2150 | 10 | 13 |
| 22 | 2250 | 11 | 14 |
| 23 | 2350 | 11 | 14 |
| 24 | 2450 | 12 | 15 |
| 25 | 2550 | 12 | 15 |
| 26 | 2650 | 13 | 16 |
| 27 | 2750 | 13 | 16 |
| 28 | 2850 | 14 | 17 |
| 29 | 2950 | 14 | 17 |
| 30 | 3050 | 15 | 18 |
| 31 | 3150 | 15 | 18 |


##### Solar Power:
Nominal base output is ~136 (8 powered tiles/solar tile). Formula appears to be:\
`(Random % (100 - humid) // 10) + 5` where % is the mod operator.

Inspection of a game showed that the minimum output was 0, the maximum output was 190 and the average was 136.

### Disasters:


### Game Models:
#### Crime Model:
Total amount of crime is stored in MISC as Crime Count. This is the sum of all values in XCRM.\
For purposes of display in the graph windows, 1 point in the window is 3750 points of crime, rounded up.

The value stored in "Arrests" in MISC is the total number of arrests for each police station microsim.
#### Land Value Model:
Total land value is stored in MISC as Land Value. This is the sum of all the values in XVAL. For display in the bond window, the displayed value is city value * 1000. For the graph display, 1 point in the window is 3200 points of value, rounded up to the nearest whole number. \
City value seems to be land value / 2.

#### Traffic Model:
Total traffic is stored in MISC as Traffic Count. It does not appear to be the sum of all the values in XTRF.

#### Weather:
The game tracks four different variables relating to weather:\

- Heat (temperature?)
- Wind: range 0-255.
- Humidity
- The actual weather. [Weather types](https://github.com/dfloer/SC2k-docs/blob/master/sc2%20file%20spec.md#weather-type)

Reportedly, crime and the weather are linked, and weather can effect disasters as well.

#### Miscellaneous Notes

The buildings that cause NIMBY reactions are:

- Gas Power Plant
- Oil Power Plant
- Nuclear Power
- Coal Power
- Prison
- Water Treatment Plant

Churches appear every 5000 people, without arcos.

Cheats:

- `joke`
- `gilmartin`
- `priscilla`
- `noah`
- `iamacheat`
- `cass`
- `fund`
- `newhouse`
  - The above cheat doesn't work, because when 'n' is typed, the game starts looking for noah, and never reaches this.
