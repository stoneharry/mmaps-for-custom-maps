# Generating correct pathfinding data for maps
## Introduction

This guide describes how to generate and debug map, vmap, and mmap data correctly for maps using TrinityCore, patch 3.3.5.

## Glossary

| Term  | Definition |
| ------------- | ------------- |
| maps | Contains basic data to calculate the games heightmap, underwater areas, fatigue areas, area information, etc.  |
| vmaps | Vector maps - used to determine line of sight information, visibility between objects. |
| mmaps | Movement maps - used for pathfinding. |

## Generating the data

TrinityCore provides tools for extracting, assembling, and building the data required. The tools are not designed to process custom data, so you must place your data emulating the Blizzlike layout:

- Place all your map data in: `Data\patch-4.MPQ`
- Place all your DBC (DBfilesClient) data in: `Data\{locale}\patch-{locale}-4.MPQ`, e.g: `Data\enUS\patch-enUS-4.mpq`

You must run the tools in order. You should run them from command prompt or powershell such that you can review the output of the programs. The map and vmap tools must be run from your WoW directory.

### Map Extractor

First run the `mapextractor.exe`. This tool also extracts DBC and Camera information, but only the map data is required for vmaps and mmaps.

Watch the map extractor tool output to ensure it does pick up and process any custom maps you have.

This will produce the `maps` directory.

### VMap extractor

Next run the `vmap4extractor.exe` and then the `vmap4assembler.exe` tools. Again, ensure it picks up and processes your custom maps by reviewing the logs.

This will produce the `vmaps` output directory. It also produces the `buildings` directory, but buildings is temporary data that can be deleted.

### MMap extractor

Next run the `mmaps_generator.exe`. This takes the map and vmap data as inputs and generates the `mmaps` directory.

This program has a lot of possible arguments that can be provided: https://github.com/TrinityCore/TrinityCore/tree/3.3.5/src/tools/mmaps_generator/Info

There are some hardcoded configurations for Blizzlike maps within the code. You may need to play with the settings for your custom map, but this is not recommended unless you know what you are doing. These configurations were only reached after extensive testing.
- https://github.com/TrinityCore/TrinityCore/blob/3.3.5/src/tools/mmaps_generator/MapBuilder.cpp#L1116

The tool also has a internal version that it stamps generated mmap data with. When you run the tool, if the data already exists with the current version then it will skip that map. This is designed such that you can run the tool each emulator update and it will only generate mmap data that has changed, but also means it is harder to iterate on a custom map being designed. You will need to delete existing mmap data for your custom map if you want to regenerate the pathfinding with map edits.

Also note that the tool has been reported to not support map IDs that are > 999.

## Debugging issues

## Maps Issues

Typically maps data should extract fine. The only thing to be wary of is checking if it does actually detect your custom map, because if it is missed at this stage then all the downstream tools will also not contain it.

## VMaps Issues

We only typically see issues here if the WMO / M2 cannot be parsed for some reason.

## MMaps Issues

This is where we frequently see issues. Movement maps can be generated vastly differently depending on the map and tool version. The settings that can be configured can also effect how it is generated.

The best way to debug movement map data is with the use of a tool called Recast Debug. However you can also:
- Use the command `.gps` to check maps, vmaps, and mmaps data exists for your current location.
- Use `.mmap` commands to debug pathfinding in game.

### Recast Debug

Recast debug allows us to view and test the pathfinding mesh from outside of WoW.

I have attached a version that has TC support to this repository.

You need to run the mmaps tool with the `--debugOutput true` argument to generate the debug data for Recast Debug.

Inside the Recast Debug `Meshes` folder you need to provide:
- The `.map` file
- The `.mmap` file
- The `.dmesh` file
- The `.mesh` file
- The `.mmtile` file
- The `.pmesh` file
- The `.mmtile` file
- The `.obj` file

![MeshesFolderScreenshot](https://i.imgur.com/Iv4NsHa.png)

When opening Recast Debug, on the rop right under `Sample` select `Tile Mesh`. TrinityCore only supports Tile Mesh.

Under `Input Mesh` select the `.obj` for the tile you want to debug.

Ensure `TrinityCore Configs` is selected.

You need to reselect the obj in Tile Mesh after enabling TrinityCore configs, otherwise the white hitbox is not 533.33f square as that is calculated only on load (it will use the obj boundaries instead).

On the left panel, select `Test Navmesh`.

Finally, on the bottom right, select `Load mmtile`.

![LoadedMmtile](https://i.imgur.com/dNbxS5t.png)

Everything within the white box is the tile. As long as your meshes folder contains neighbouring tiles, they will also be displayed outside of the white box for completeness. In the above screenshot we can see the WMO spans multiple tiles (ADTs).

The red area indicates where pathfinding has successfully been loaded. We can right and left click to preview how pathfiding would be calculated between two locations.

![PathfindingPreview](https://i.imgur.com/8g9mMm7.jpeg)
