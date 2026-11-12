<p align="center">
  <img src="https://raw.githubusercontent.com/OpenTS-Developers/.github/main/assets/opents-logo.png" alt="OpenTS" width="512">
</p>

<p align="center">
  <a href="https://github.com/OpenTS-Developers/OpenTS/releases"><img src="https://img.shields.io/github/downloads/OpenTS-Developers/OpenTS/total?label=downloads" alt="Downloads"></a>
  <a href="https://github.com/OpenTS-Developers/OpenTS/actions/workflows/engine.yml"><img src="https://github.com/OpenTS-Developers/OpenTS/actions/workflows/engine.yml/badge.svg" alt="Engine build"></a>
  <a href="https://opents-developers.github.io/OpenTS/"><img src="https://github.com/OpenTS-Developers/OpenTS/actions/workflows/manual-pages.yml/badge.svg" alt="Manual"></a>
  <a href="https://www.patreon.com/c/ZivDero"><img src="https://img.shields.io/badge/Patreon-ZivDero-F96854?logo=patreon&logoColor=white" alt="Patreon"></a>
</p>

# OpenTS

OpenTS is a community-led, open-source reconstruction of *Command & Conquer:
Tiberian Sun*. Instead of patching or extending the retail executable, it
rebuilds the engine as a standalone program. For more information about
this project, please visit the OpenTS Repository

# OpenTS-macos-ios

Is a fork from the original OpenTS project with a build process for Mac and iOS. 

This fork carries native Apple Silicon builds of that engine for **macOS and
iOS**. The iPad build is played by touch: it has no mouse pointer, a gesture
layer in place of one, and an on-screen indicator that reports the mode the
cursor would otherwise have shown.

For more information about the project please visit the fork's 
repository at [here](https://github.com/msnow345/OpenTS-macOS-iOS).

## ABOUT THIS FORK

This fork is aimed at personal use and I cannot provide any support whatsoever
about running on any device other than mine. The goal was to modernize the 
Tiberian Sun engine with Quality-of-Life improvements from other games like 
Red Alert 2 and C&C Remastered. 

Some of the changes implemented are:
* Fixes UI and drawing bugs (like the turret on the Titan
* High-res voxel rendering (all credit to the Tiberian Sun Recompilation project)
* Different production queues for Factories, Support/Defense, Infantry and Vehicles like in Red Alert 2
* In-game (single player only) cheats menu (No damage, reveal map, money, build anywhere)
* Restore some missing animations and art

To-do
* Implement battlefield zooming while keeping the UI scaled correctly. 
Currently playing at high resolutions makes unit control very difficult.

## AI DISCLAIMER

100% of the code for the modifications has been written by CODEX from OpenAI. 
I do not take any credit for it other than guidance and review to
ensure the features behaves as intended while keeping the essence of a game I love.


## Running on macOS

This fork builds and runs the engine natively on Apple Silicon. The game data
still comes from a copy of Tiberian Sun you own.

Install the tools:

```bash
brew install cmake ninja
brew install --cask steamcmd
```

Fetch the game data from your own Steam account:

```bash
./scripts/get-assets.sh <your_steam_username>
```

The script downloads app 2229880 into `Run/`, skips the Windows executables the
engine replaces, and checks that the archives startup needs actually arrived.
Steam Guard prompts for a code on first login. **Quit the Steam desktop client
first**: steamcmd shares its data directory, and a running client holds a lock
that makes steamcmd hang after "Verifying installation..." with no error. Set
`OPENTS_GAME_DIR` to put the data somewhere other than `Run/`.

Build the engine:

```bash
git submodule update --init --recursive
cmake -S . -B build/native -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DOPENTS_EXPERIMENTAL_NATIVE=ON
cmake --build build/native --target OpenTS
```

Build the `OpenTS` target rather than everything: the C++ test harnesses pass
MSVC-only flags and link `kernel32`, so they do not configure here.

Run it:

```bash
build/native/bin/Game -DATADIR=Run -USERDIR=build/native/user
```

`-USERDIR` is where saves, `SUN.INI` and logs are written, so pointing it at a
scratch directory leaves an existing install untouched. Full screen is a
setting in the display options screen.

Windows supplies the window, message loop and cursor itself; every other
platform gets them from `platform/win32compat`, which serves the Win32 surface
the engine is written against out of [SDL](https://github.com/libsdl-org/SDL).
[Building OpenTS](docs/BUILDING.md) covers the build in full.

## Running on iOS

The same engine runs on iPad. It needs an Apple developer account to sign with
and a device paired to the Mac; a paid account provisions for a year, a free
one for seven days.

Copy the signing template and fill it in. The real file is git-ignored, so no
team identifier, signing identity or device UDID is ever committed:

```bash
cp scripts/build/ios/ios-signing.env.example scripts/build/ios/ios-signing.env
```

It names `OPENTS_IOS_TEAM_ID`, `OPENTS_IOS_CODESIGN_IDENTITY` (use an **Apple
Development** identity — a Distribution one will not install directly),
`OPENTS_IOS_BUNDLE_ID` and `OPENTS_IOS_DEVICE`. Then build, sign, install, and
copy the game data into the app's own container:

```bash
./scripts/build/ios/package-ios.sh --clean --install --push-data --launch
```

`--push-data` is only needed the first time, or when the data changes. After
that `--resign-only --install` is the fast loop. **Unlock the device first** —
a locked iPad refuses the install.

`docs/BUILDING.md` covers the presets, the on-device data layout and how the
bundle is put together.

### Playing by touch

There is no mouse pointer on iOS, so the cursor's two jobs are split: gestures
do what the pointer did, and an indicator reports what its shape used to say.

| Gesture | Action |
| --- | --- |
| Tap | Select, or give the order under your finger |
| Drag with one finger | Rubber-band select |
| Drag with two fingers | Pan the map, 1:1, with a coasting flick |
| Press and hold | Cancel the current mode, then deselect |
| Tap during a movie | Skip it |

Nothing is dispatched until a gesture is known, so a second finger arriving
turns a pending tap into a pan without leaving a stray click behind — which in
this game would otherwise issue an order or place a building. Pinch zoom is
deliberately absent: the tactical view magnifies an already-drawn frame rather
than revealing more map, so pinching out would only blur it.

An icon in the bottom left corner shows the game's own cursor artwork,
animated, whenever it is saying something a finger cannot: sell, repair, power,
waypoint and super weapon modes while they are armed, and a unit that is
standing somewhere it can deploy. It stays empty the rest of the time, because
with no hover the shapes that merely describe what is under the pointer can
only appear after the tap that already acted.

The frame is laid out for the device's own display rather than a fixed
resolution, and the frame rate is capped by `MaxFrameRate` in the `[Video]`
section of `SUN.INI`.

## Documentation

The [OpenTS manual](https://opents-developers.github.io/OpenTS/) documents
setup, runtime behavior, INI configuration, mapping, and source-level
internals.

## License and acknowledgements

OpenTS is licensed under the GNU General Public License, version 3 or later.
Material derived from Electronic Arts source remains subject to the additional
GPL Section 7 terms in [LICENSE.md](LICENSE.md).
[Third-party notices](THIRD_PARTY_NOTICES.md) identify bundled dependencies
and their licenses.

[ACKNOWLEDGEMENTS.md](ACKNOWLEDGEMENTS.md) thanks the people, projects, and
communities whose work made OpenTS possible.
