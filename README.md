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

Furthermore, the tool has a lot of hardcoded configurations for Blizzlike maps within the code. You may need to play with the settings for your custom map.
- https://github.com/TrinityCore/TrinityCore/blob/3.3.5/src/tools/mmaps_generator/MapBuilder.cpp#L1116

The tool also has a internal version that it stamps generated mmap data with. When you run the tool, if the data already exists with the current version then it will skip that map. This is designed such that you can run the tool each emulator update and it will only generate mmap data that has changed, but also means it is harder to iterate on a custom map being designed. You will need to delete existing mmap data for your custom map if you want to regenerate the pathfinding with map edits.

## Debugging issues

## Maps Issues

## VMaps Issues

## MMaps Issues

### Recast Debug


