# SC2k-docs
Unofficial documentation related to the implementation of Maxis' game, SimCity 2000.
## License:
[![licensebuttons by-sa](https://licensebuttons.net/l/by-sa/3.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0)\
This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).
## Contents:
- `sc2 file spec.md`: specifications on the .sc2 file format.
- `sprite data spec.md`: Specifications of the data files used to store the various building, terrain, road and other sprites in the game with a section for the additional specifications for MIF files.
- `text data spec.md`: Specifications for the newspaper and other text data files for the game. The newspaper specification is considered a work in progress as it can parse all of the text, but very little around how stories are composed has been determined.
- `simulation spec.md`: Specifications for how the actual simulation works. Examples: how the power or water system works, how traffic works.

## Status:
The sc2 file specifications are largely complete. Sprite parsing is complete. The text data is useful to getting raw data for the newspapers, but not generating newspaper stories, while the rest of the text format is simple and captured in the documents.

The simulation spec is a start at cataloguing all of the information about the internals of the game.

## I'd like to help:
Great! Open a Pull Request with a correction or additional information. If it's spelling or otherwise simple, it should be merged right away.

For things more complex, please add supporting evidence, such as screenshots from the game, discussion of testing methodology, etc. Basically, enough that someone else can reproduce the results. Be wary of the manuals and other information floating around online, they're not always right on how the game actually works internally.

Opening an issue is also a good place to get started, and allows discussion while figuring whatever out.