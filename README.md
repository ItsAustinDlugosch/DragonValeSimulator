# DragonVale Breeding Simulator

A Swift command-line project for simulating DragonVale breeding outcomes.

This was built as a personal tool for making better in-game decisions. The goal was to model the game's breeding system closely enough to compare parent combinations, simulate possible outputs, and reason about which breeding choices were most likely to produce useful dragons. The dataset in this repo was reverse engineered from game information and represented locally as structured Dragonarium data.

## What It Does

The project models dragons, breeding requirements, cave settings, and result probabilities for DragonVale breeding.

At a high level, the simulator:

- Loads Dragonarium data from `json/dragonarium.json` or `csv/dragonarium.csv`.
- Represents each dragon with rarity, elements, breeding availability, breeding time, clone chances, earnings, collections, rift data, and special requirements.
- Combines two parent `DragonInstance` values into a `BreedComponents` object.
- Adds environmental context such as cave type, time of day, weather, and rift alignment.
- Finds all dragons whose requirements are satisfied by that breeding combination.
- Calculates output probabilities across clone results, fixed-percentage results, and fluctuating results.
- Includes an optimization helper for combining target/favorable results into higher-value breeding requirements.

The sample `main.swift` initializes the Dragonarium, configures the simulator for a normal cave at night, and attempts to simulate breeding `Whirlwind` with `Khamsin`.

## How The Simulation Works

The core flow lives in `BreedSimulator` and `BreedSimulation`.

`BreedSimulator.combineBreedComponents` looks up both parent dragons in the Dragonarium and builds the effective breeding input:

- Parent elements are combined, including repeated element counts where requirements need duplicates.
- Parent dragon identities are tracked as `DragonInstance` values.
- Rift caves add rift elements to the combination.
- Rift traits, rift alignment, cave type, time of day, and weather become special breed requirements.
- Opposite primary elements are recognized for primary dragon outcomes.

`BreedSimulation` then evaluates the Dragonarium against those components:

- Fixed dragons are included when their full breed requirements are satisfied.
- Hybrid fluctuating dragons are included when their required primary elements are present.
- Primary fluctuating dragons are included for matching opposite-element pairs.
- Clone outcomes are calculated separately using cave-specific single-clone and double-clone rates.
- Fixed probabilities are applied first, with a cap so clone plus fixed results do not exceed 99%.
- Any remaining probability is split evenly across fluctuating outcomes.

This mirrors the project's reverse-engineering approach: encode the discovered rules as composable requirements, then let the simulator derive the result pool from data rather than hard-coding every parent pair.

## Project Structure

```text
.
|-- BreedSimulator/     # Cave settings, breeding combination logic, probability simulation
|-- Dragon/             # Dragon, elements, rarity, time, availability, and requirement models
|-- Dragonarium/        # Dragonarium aggregate and collection models
|-- Extensions/         # Small Set and Array helpers used by requirement merging
|-- FileManagers/       # JSON and CSV loading/encoding helpers
|-- Protocols/          # Shared file-writing helper protocol
|-- csv/                # Source Dragonarium CSV data
|-- json/               # Encoded Dragonarium JSON data
`-- main.swift          # Current command-line entry point / sample simulation
```

## Running Locally

This repo does not currently include a Swift Package Manager `Package.swift` file or an Xcode project. It can be compiled as loose Swift source files.

From the repo root:

```bash
mkdir -p /tmp/dragonvale-swift-module-cache /tmp/dragonvale-build
swiftc -module-cache-path /tmp/dragonvale-swift-module-cache \
  main.swift \
  BreedSimulator/*.swift \
  Dragon/*.swift \
  Dragonarium/*.swift \
  Extensions/*.swift \
  FileManagers/*.swift \
  Protocols/*.swift \
  -o /tmp/dragonvale-build/dragonvale-sim
```

Then run:

```bash
/tmp/dragonvale-build/dragonvale-sim
```

Run it from the repo root because `main.swift` loads `./json/dragonarium.json` with a relative path.

## Current Status

The codebase is a project snapshot rather than a polished application.

Verified in this environment with Swift 6.2.1:

- The source files compile with warnings about a few local variables that could be `let`.
- The compiled sample currently crashes while decoding `json/dragonarium.json`.
- The crash happens because `BreedInformation` requires a `breed_type` field, but the bundled JSON data does not contain that key.
- The CSV importer also appears to expect a schema that has drifted from the checked-in CSV header.

In other words, the domain model and simulator logic are present, but the data schema and loader code need to be brought back into sync before the sample command-line simulation will run successfully end to end.

## Data Notes

The Dragonarium data is stored in two forms:

- `csv/dragonarium.csv`: tabular source data with dragon names, rarity, requirements, elements, times, percentages, collection membership, and economy fields.
- `json/dragonarium.json`: structured encoded data consumed by the current `main.swift`.

The data reflects a reverse-engineered understanding of DragonVale's breeding behavior. It is intended for analysis, simulation, and educational/personal use.

## Possible Next Improvements

- Add a `Package.swift` manifest so the project can be built with `swift build`.
- Reconcile the CSV/JSON schemas with the current Swift models, especially `breed_type`.
- Replace the hard-coded sample in `main.swift` with command-line arguments for parent dragons and cave settings.
- Add tests for requirement matching, special-condition merging, clone probabilities, and percentage normalization.
- Add a small report mode for ranking parent combinations by target-dragon or favorable-result probability.

## Disclaimer

This is an unofficial educational and personal project. It is not affiliated with DragonVale or its publishers/developers.
