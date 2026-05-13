# HexedUT2k4 - Map Previews

This repository is a collection of packages providing an alternative way to show map previews in the map vote menu when the map is not locally installed.
All packages are designed to work together with [HexedVOTE](https://github.com/hexengraf/HexedUT2k4).

Current packages:
* HexedPreviewsBASE: contains all CBP maps plus a few "semi-official" and notable maps. It also associates "fixed" versions of official maps with the original previews.
* HexedPreviewsUCMP: contains all UCMP maps.

## Configuration

First you need to install [HexedVOTE](https://github.com/hexengraf/HexedUT2k4).
Next, decide which preview packages you want in your server.
Let's say you only want HexedPreviewsBASE. You need to add it to your `ServerPackages` and to `MapPreviewLoaders` under the HexedVOTE section:
```ini
[Engine.GameEngine]
; Add it at the end of the ServerPackages list
ServerPackages=HexedPreviewsBASEv1

[HexedVOTEv7.MutHexedVOTE]
; All packages from this repository use HxLoader as the Loader name, but custom packages might have different names.
MapPreviewLoaders=HexedPreviewsBASEv1.HxLoader
```

## Contributing (or creating your own packages)

Contributions to this repository are greatly welcomed! We can create new packages to group previews.
Unfortunately only linux is supported with the current build system.
If you find a way to support Windows without breaking compatibility with the current build system, feel free to open a PR.

> [!WARNING]
> This repository uses symbolic links, so if you're on Windows you might need to change a git configuration to properly clone with symbolic links.

### How to add a new package

The fastest way to get started is to copy the folder of an existing package and start renaming things.

The main class is the loader, by default named `HxLoader`. It imports another file with all the boilerplate code:
```UnrealScript
class HxLoader extends Object
    abstract;

#include Classes\Include\HxLoader.uci
```

The folder `Classes/Include` is a symbolic link to `Common/Include`, so make sure it is properly set up.

Inside `HxLoader`, you want to populate the `Entries` array inside the default properties block:
```UnrealScript
defaultproperties
{
    Entries(0)=(MapName="AS-Confexia",Preview=Material'HxPreviewsBASE.AS_Confexia')
    Entries(1)=(MapName="BR-CBP1-BreakLimit2004",Preview=Material'HxPreviewsCBP.BR_BreakLimit2004')
    Entries(2)=(MapName="BR-CBP2-Aquarius",Preview=Material'HxPreviewsCBP.BR_Aquarius')
    // ...
}
```
Each entry associates a map name with a corresponding preview material.
You can have multiple entries with different map names associated with the same preview.

If you have custom maps that should use the previews of an official map, there is a separate array for that:
```UnrealScript
defaultproperties
{
    OfficialEquivalents(0)=(MapName="AS-CBP2-Thrust",OfficialName="AS-BP2-Thrust")
    OfficialEquivalents(1)=(MapName="CTF-CBP2-Pistola",OfficialName="CTF-BP2-Pistola")
    OfficialEquivalents(2)=(MapName="DM-1on1-Roughinery-FE",OfficialName="DM-1on1-Roughinery")
    // ...
}
```
In this array, each entry associates a custom map name with an official map name, so the loader can find the official preview for it.
It is highly recommended to add `OfficialEquivalents` entries to the `HexedPreviewsBASE` package (please!).

> [!WARNING]
> Both `Entries` and `OfficialEquivalents` **MUST** be defined in lexicographic order.
> If you get the order wrong, some maps may not be found and will show "No Preview Available" instead.

The folder `Textures` is a symbolic link to `Common/Textures`. This is where all screenshots should be saved.

Other classes inside the package are bundles of preview materials. For instance, `HxPreviewsCBP` has all CBP previews.
There are two parts to it. First, you need to use `exec` directives to import the screenshots, e.g:
```UnrealScript
#exec texture Import File=Textures\BR_CBP2_Aquarius_p1.dds Name="BR_CBP2_Aquarius_p1" Mips=Off DXT=1
#exec texture Import File=Textures\BR_CBP2_Aquarius_p2.dds Name="BR_CBP2_Aquarius_p2" Mips=Off DXT=1
#exec texture Import File=Textures\BR_CBP2_Aquarius_p3.dds Name="BR_CBP2_Aquarius_p3" Mips=Off DXT=1
```

Make sure to disable MipMaps and use DXT1 compression to get the smallest package size possible.

The second part is the definition of the `MaterialSequence` using the screenshots, which should be inside the default properties block, e.g:
```UnrealScript
defaultproperties
{
    Begin Object Class=MaterialSequence Name=BR_Aquarius
        SequenceItems(0)=(Material=Texture'BR_CBP2_Aquarius_p1',Time=2.0,Action=MSA_ShowMaterial)
        SequenceItems(1)=(Material=Texture'BR_CBP2_Aquarius_p2',Time=0.5,Action=MSA_FadeToMaterial)
        SequenceItems(2)=(Material=Texture'BR_CBP2_Aquarius_p2',Time=2.0,Action=MSA_ShowMaterial)
        SequenceItems(3)=(Material=Texture'BR_CBP2_Aquarius_p3',Time=0.5,Action=MSA_FadeToMaterial)
        SequenceItems(4)=(Material=Texture'BR_CBP2_Aquarius_p3',Time=2.0,Action=MSA_ShowMaterial)
        SequenceItems(5)=(Material=Texture'BR_CBP2_Aquarius_p1',Time=0.5,Action=MSA_FadeToMaterial)
        TotalTime=7.5
        Loop=True
        Paused=False
    End Object
}
```

The fade and show time can be further adjusted to get a result closer to the original preview.
You need to manually specify the sum of all times in `TotalTime` otherwise the sequence will not work correctly.

### How to prepare screenshots

You have two alternatives: create your own screenshots or extract the original screenshots from the map.
You can probably find tutorials online about how to prepare screenshots for map previews.

There are two things you must follow before submitting your screenshots to this repository:
* The screenshots must be 512x256 pixels.
* Use DDS format with DXT1 compression and MipMaps disabled.

A lot of the screenshots directly extract from maps have MipMaps (it was useful for really old hardware), but the focus here is minimizing size.

## Known issues

* DM-CBP2-TensileSteel is missing 2 screenshots. For some unknown reason I wasn't able to extract textures from Tensile.utx, the two screenshots I got from UnrealArchive instead.
* AS-UCMP2-Cruciatus, AS-UCMP3-IslandStrike, and ONS-UCMP-ABC have screenshots not properly formatted. I wasn't able to open them with Gimp to re-export.

## Copyright notice

This project does not make any copyright claims over map screenshots extracted from the original map textures, all rights belong to the original author(s).
If you want your screenshot(s) removed from this repository, please open an issue.

The MIT license only applies to the UnrealScript portions of this project.
