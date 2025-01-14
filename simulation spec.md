# Simulations Specifications

## About these specifications:

At this point, these specifications are largely structured as notes rather an implementable specification due to the very incomplete nature of them. Written for the Windows 95 Special Edition version.

### Starting a New City

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
- Invention dates are calculated from the base year they can be [invented](sc2%20file%20spec.md#technology-discovery-years) with an additional 0 to 19 years randomly added.

### Terrain Generation

#### Terrain Generation settings

| Name | Range | Default |
| --- | --- | --- |
| Coast | 0 or 1 | 0 |
| River | 0 or 1 | 0 |
| Hills | 0 to 47 | 12 |
| Water | 0 to 47 | 5 |
| Trees | 0 to 47 | 15 |

Selecting Coast also allow salt-water to be placed on the map.

### Water Production

Water production is measured in the number of tiles it will serve, not the gallons displayed in game.

There's a display bug where a single water pump in a city with no consumer makes the water shortage check fail, this does not mean that the city doesn't need water. This has been called the "Phantom Water Pump Trick", but it's not useful to **actually** get rid of the need for a water system.

#### Water Pumps

Water pump production is base on the sea level, how many fresh water tiles are bordering it (8 maximum) and how much precpitation is happening.

The formula the game uses appears to be: `sea level * 5 + fresh water tiles * 10 + precipitation / 2`

To convert to gallons of water, round this value down and multiply by 720.

Example from a city with a sea-level of 4, 4 fresh water tiles around the pump and 14mm of rain: \
4 * 5 + 4 + 10 + 14 / 2 = 67.0\
Game shows 48,240 gallons/month (=67 * 720).

#### Desalinization

The formula appears to be `salt water tiles * 20`.

Note that as multi-tile buildings are calculated for each tile in a building, this means each tile needs to be calculated independently and summed up.

This means that a desalinization plant surrounded on all sides by salt water will produce 640 files worth of water.

How is this calculated?

There are 9 tiles of the desalinization plant. 4 tiles border 3 saltwater tiles (the middle outside tiles), and 4 tiles border 5 saltwater tiles (the corner tiles), with one tile (the inside middle tile) bordering no saltwater tiles. This gives us `4 * 5 + 4 * 3 + 1 * 0 = 32` tiles. Which, multiplied by 20 = 640.

Another example, with a power line going to the plant in the middle of one side. We have one less saltwater tile, so the calculation is now based on: 2 tiles of 5 (the two outside corners away from the power line), 3 tiles of 3 (the three edges without the power line), 2 tiles of 4 (the two corners near the power line), and finally one tile with two bordering salt water tiles (the tile the power line lands on): This gives `2 * 5 + 3 * 3 + 2 * 4 + 1 * 2 = 29`, and the game shows 580 tiles of water produced (29 * 20 is 580).

Note that the internal game calculation goes a tile at a time, but I grouped tiles together for illustrative purposes.

#### Water Towers

Unknown exactly, but water towers appear to store a maximum of 400 tiles of water, in 100 tile increments. Water towers only store excess production from pumps, and "discharge" water only when production is lower than demand.

#### Water Treatment Plants

There's a bonus for having enough water treatment plants.

One water treatment plant is needed for every 2,000 tiles in a city.

### Power Plants

Methodology to determine: determined by building a city with low density zones and calculating how many tiles each power plant could power. Note that these numbers include the power plant, as it seems to require power to operate.

| Plant | Nominal Output (MW) | Actual Output (tiles) | Efficiency |
|---|---|---|---|
| Coal | 200 | 704 | 44 |
| Hydroelectric | 20 | 40 | 40 |
| Oil | 220 | 768 | 48 |
| Gas | 50 | 176 | 11 |
| Nuclear | 500 | 1776 | 111 |
| Microwave | 1600 | 5680 | 355 |
| Fusion | 2500 | 8880 | 555 |

Efficiency is simply the number of tiles a plant can power / the number of tiles the plant takes up. So for the Coal plant this is 704 / 16 = 44. This is not part of the simulation, just included for illustrative purposes.

The energy saving ordinance appears to save 1/12 (~8.33%) power.

#### Wind Power

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

#### Solar Power

Nominal base output is ~136 (8 powered tiles/solar tile). Formula appears to be:\
`(Random % (100 - humid) // 10) + 5` where % is the mod operator.

Inspection of a game showed that the minimum output was 0, the maximum output was 190 and the average was 136.

### Disasters

Disasters won't start too early in the game, dependend on the game's difficulty (this might be incorrect):

- On easy, no disaster for the first three months
- On medium, no disasters for the first two months
- On hard, no disaster for the first month.

#### Disaster start conditions

Most disasters have a random start component

- Hurricane:
  - Weather has to be `0x0A` (Hurricane).
  - Has to have coastal terrain (from map generation).
- Tornado:
  - Weather has to be `0x0B` (Tornado).
- Riot:
  - Unemployment is 10 or more.
  - Heat is over 170.
  - Population is over 30,000 (without arcos).
- Meltdown:
  - Nuclear power plant needs to exist.
- Microwave:
  - Microsave power plant needs to exist.
- Air Crash:
  - There have to be runway tiles.

### Game Models

#### Crime Model

Total amount of crime is stored in MISC as Crime Count. This is the sum of all values in XCRM.\
For purposes of display in the graph windows, 1 point in the window is 3750 points of crime, rounded up.

The value stored in "Arrests" in MISC is the total number of arrests for each police station microsim.

#### Land Value Model

Total land value is stored in MISC as Land Value. This is the sum of all the values in XVAL. For display in the bond window, the displayed value is `city value * 1000`. For the graph display, 1 point in the window is 3200 points of value, rounded up to the nearest whole number. \
City value seems to be `land value / 2`.

#### Traffic Model

Total traffic is stored in MISC as Traffic Count. It does not appear to be the sum of all the values in XTRF.

#### Weather

The game tracks four different variables relating to weather:

- Heat (temperature). In the newspaper, this degrees F, minus 100. So 186 in game is 86°F in the newspaper
- Wind: range 0-255. This appears to nominally be in miles per hour.
- Humidity: Appears to be rain, newspaper shows this as mm (not inches?) of rain.
- The actual weather. [Weather types](sc2%20file%20spec.md#weather-type)

Reportedly, crime and the weather are linked, and weather can effect disasters as well.

#### Budget

For bonds, the game appears to display decimal rates rounded down in the budget window, but calculates costs based on the actual rate.

The bond sum rate may be used to determine credit rating, in conjunction with the city's land value.

##### Taxes

Tax revenue is calculated as `tax rate * population / 75`. Land value does not appear to affect tax revenue, but reportedly it was _intended_ to affect tax revenue.

Difficulty appears to change how sensitive sims are to taxes.

##### Ordinances

Ordinances appear to have effects on zone demand, taxes raised and other effects.

Notes:

- This is still incomplete, and ?'s are used in place of unknown values.
- ?? means unknown if effect exists.
- Cost is given of that % of tax revenue it costs.
- Demands changes were determined experimentally.
- Costs were estimated based on looking at the cost versus tax revenue.
- LE/EQ boosts are unknown, but seem to exist.
- Parking fines may not actually impact traffic.

| Name | Demand Effect | Tax Effect | Other Effect(s) | Cost |
| --- | --- | --- | --- | --- |
| 1% Sales Tax | -1% Commercial | +1% Commercial Revenue | - | 0% |
| 1% Income Tax | -1% Residential | +1% Residential Revenue | - | 0% |
| Legalized Gambline | - | +2% Commercial Revenue | (?) crime increase | 0% |
| Parking Fines | - | +0.5% Residential Revenue | (??) traffic change | 0% |
| Pro-Reading Campaign | - | - | ?? Increased EQ | 1/6% Res |
| Anti-Drug Campaign | - | - | ?? Increased LE | 1/5% Res |
| CPR Training | - | - | ?? Increased LE | 1/6% Res |
| Neighborhood Watch | - | - | - | 1/3% Res |
| Energy Conservation | - | - | Power usage drops by 1/12 (~8.333%) | 1/2% All |
| Nuclear Free Zone | - | - | No Nuclear Power plants can be built. | 0% |
| Homeless Shelter | +1% Commercial | - | - | 0.5% Res |
| Pollution Controls | -1% Industrial | - | Less pollution | 1% Ind |
| Volunteer Fire Dept. | - | - | ?? Increased fire power. | 1/3% Res |
| Public Smoking Ban | - | - | ?? Increased LE | 1/6% Res |
| Free Clinics | - | - | ?? Increased LE | 1/2% Res |
| Junior Sports | - | - | ?? Increased LE/EQ | 1/4% Res |
| Tourist Advertising | +1% Commercial | - | - | 1% Comm |
| Business Advertising | +1% Industrial | - | - | 1% Ind |
| City Beautification | +1% Residential | - | - | 1/4% Res |
| Annual Carnival | +1% Commercial | - | - | 1/3% Comm |

#### Miscellaneous Notes

The buildings that cause NIMBY reactions are:

- Gas Power Plant
- Oil Power Plant
- Nuclear Power
- Coal Power
- Prison
- Water Treatment Plant

Churches appear every 5000 people, without arcos.

##### Citizen Demands

Citizens demand services at certain points. Unless otherwise noted, population counts are without arcos.

- Schools are every 20,000 sims.
- Fire stations are every 20,000 sims.
- Hospitals are every 25,000 sims.
- More water capacity is needed if utilization reaches 98%.
- More power capacity is needed if utilization reaches 98%.
- Churches are automatically built every 2,500 sims, but only replace an existing 2x2 residential building.
- Residential recreation demand only crops up after 10,000 population.
  - A marina increases the residential demand cap by 9,000 total.
  - A big park increases the residential demand cap by 3,000.
  - Zoos and stadiums each increase the residential demand cap by 16,000.
- Airport demand is every 10,000 commercial population (potentially only the runway cross counts).
- A road connection satisfied 2,000 commercial population demand.
- Seaport demand is every 10,000 industrial population (only the pier counts).
- A rail connection also satisfies 10,000 industrial population.
- A highway connection also satisfies 10,000 industrial population.

## Cheats

- `joke`
- `gilmartin`
- `priscilla`
- `noah`
- `iamacheat`
- `cass`
- `fund`
- `newhouse`
  - The above cheat doesn't work, because when 'n' is typed, the game starts looking for noah, and never reaches this.
