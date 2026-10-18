---
title: Visual Studio template
parent: Getting started
nav_order: 9
---

# Visual Studio template

Visual Studio 2022 can create the mod project for you. The template makes the same project as [Your first mod](first-mod.md), with your project name filled in. You do not copy or rename files by hand. Once the project exists, both paths are the same.

## Install the template

You need Visual Studio 2022, at least the Community edition. You also need the same files as on the [install page](../../install.md): the game, [BepInEx](../../glossary.md#bepinex) and the [framework](../../glossary.md#framework), plus the .NET 10 SDK.

1. Download `ObenseuerModTemplate.zip` from the [releases page](https://github.com/NomadsReach/Obenseuer-Modding-Framework/releases), the same page you got the framework from.
2. Copy the zip into the folder where Visual Studio keeps your own project templates. To open that folder, paste this line into the address bar of File Explorer and press Enter:

```
%USERPROFILE%\Documents\Visual Studio 2022\Templates\ProjectTemplates
```

3. Restart Visual Studio. It reads that folder when it starts, so a zip copied while it is running is not listed yet.

Do not unzip the zip and do not move files into it: its `MyTemplate.vstemplate` file has to stay at the root of the zip.

The template is installed. Next, create the project.

## Create the project

1. Choose Create a new project.
2. Type `Obenseuer` in the search box, pick Obenseuer Mod (Framework), then Next.
3. Type the project name and a location. Use a plain name such as `CoolMod`. The name becomes the class name and the namespace, so keep it to letters, digits and underscores.
4. Choose Create. Visual Studio makes a folder named after the project, holding `CoolMod.csproj`, `Plugin.cs`, `Directory.Build.props` and `README.txt`.

The project exists. Before you build, check the mod id in `Plugin.cs`.

`Plugin.cs` is a mod that writes one log line and shows a hint once a save loads. Its `ModInfo` id is `example.<ProjectName>`. [Mod ids](../../glossary.md#mod-id) are customarily lower case, and the template cannot lower case your name, so edit the id to `example.coolmod` by hand. It has to stay unique, because the log file, the settings file and the save data are all named after it.

## Build the project

Choose Build, then Build Solution, or open a terminal in the project folder and run:

```
dotnet build -c Release
```

The last lines should say `Build succeeded.`, `0 Warning(s)` and `0 Error(s)`. The DLL is `bin\Release\net472\<ProjectName>.dll`. Nothing is copied into the game. To copy it there from Visual Studio, add `<DeployToGame>true</DeployToGame>` to the `PropertyGroup` of the `.csproj` file. Or build on the command line with `-p:DeployToGame=true`, as in [Your first mod](first-mod.md).

### Common problems

- The build cannot find the game, BepInEx or the framework: its message names the missing piece. Finish the [install page](../../install.md) and start the game once.
- The game is not in the default Steam folder: add a file named `GameDir.user.props` next to `Directory.Build.props` in the project folder. Put your own path in it, the folder that holds `Obenseuer.exe`:

```xml
<Project>
  <PropertyGroup>
    <GameDir>D:\Games\Obenseuer</GameDir>
  </PropertyGroup>
</Project>
```

## Where to go next

The walkthrough is [Your first mod](first-mod.md). The pages after it each add one feature to this same project, so keep it.
