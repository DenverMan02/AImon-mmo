# AImon MMO — Instructions for AI Coding Agents 

These instructions apply to the whole repository. Read them before making changes, then inspect the relevant source files and current Git state. The user's latest explicit direction takes precedence over assumptions in this document.

## 1\. Project overview

AImon MMO is an early-stage, Pokémon-inspired online game project. It is a separate project built with Phaser; it is **not** a Pokémon Essentials game and does not run on RPG Maker XP, Ruby, or RGSS.

The project is currently a small browser application. The existing Phaser scene is only a startup test, not the final game structure or visual design.

Important working principles:

* Do not assume an unfinished game mechanic, UI design, battle format, progression system, or server feature is finalized unless it is documented or the user confirms it.
* Do not invent major game-design decisions to fill gaps. Identify the missing decision and, when it is not blocking, make the smallest reversible implementation.
* Treat Pokémon Essentials and its add-ons as references and sources of imported resources, not as the runtime foundation of AImon.
* The project may be menu-heavy. Do not make a traditional overworld, tilemap, or top-down movement system an architectural prerequisite.

## 2\. Current technology

The current project uses:

* **Phaser 4** (phaser dependency, currently ^4.2.1)
* **Vite** (^8.3.8) for the development server and production build
* **TypeScript** (^7.0.2)
* Native ES modules (type: module)

Use Phaser 4 APIs and the installed package's TypeScript definitions. Do not copy Phaser 3 examples blindly, use Phaser AE APIs, or downgrade the project to another Phaser version.

Useful commands:

* npm install
* npm run dev
* npm run build
* npm run preview

Command purposes:

* npm run dev starts the local Vite development server.
* npm run build runs the TypeScript check and Vite production build.
* npm run preview previews the production build.
* There is no dedicated automated test script configured yet. Do not claim tests passed unless an actual test suite has been run.

Keep package-lock.json in version control. Do not update dependency versions, add frameworks, or introduce packages without a clear technical reason. In particular, React and a specific MMO backend have **not** been selected as project requirements.

## 3\. Architecture direction

The architecture is still developing. Use these boundaries as guidance when adding code; do not create empty layers or abstractions without a current need.

* src/game/: Phaser-specific scenes, rendering, animation, input, and visual game objects.
* src/ui/: reusable menus and interface components, when a separate UI layer is useful.
* src/core/: game rules and calculations that should not depend on Phaser.
* src/data/: structured game data such as Pokémon, moves, abilities, items, and type information.
* src/network/: client/server communication once the backend and protocol are designed.
* src/utils/: small, genuinely shared helpers.

### Keep game rules separate from presentation

Battle calculations and other authoritative game rules should be implemented independently of Phaser wherever practical. The goal is to let the same TypeScript logic be used by:

1. The browser client, which presents the battle.
2. A future server, which validates and resolves authoritative game actions.
3. A future headless simulator, which can run large numbers of battles without rendering them.

Do not put damage formulas, turn resolution, type matchups, or other core rules inside Phaser scenes or UI components. Avoid importing Phaser from core game-logic modules.

The server, battle simulator, CDN provider, and final UI framework are not implemented or finalized. Do not pretend they already exist or choose them without the user's direction.

## 4\. Phaser and battle UI

Phaser owns the game canvas and Phaser-rendered game presentation. Ordinary HTML/CSS may also be used where it is a better fit for menus or account-style interfaces; do not force every interface element into the canvas.

The current battle UI reference is:

* public/assets/concepts/battle ui/basic1.png
* public/assets/concepts/battle ui/basic1 explanation.txt

The accompanying notes describe the intended orientation for that concept: player side on the left, opponent side on the right, with player-side Pokémon sprites flipped horizontally to face the correct direction. The image is a layout reference, not a final-scale specification.

Pokémon Essentials' battle UI can be studied as a reference for layout, behavior, and asset conventions. Its Ruby/RGSS classes cannot be used directly in Phaser. Recreate or port the useful behavior in TypeScript and Phaser rather than trying to run the Essentials rendering engine inside this project.

## 5\. Source code conventions

For **new code and folders**, use:

* Folder and file names: lowercase kebab-case, e.g. battle-scene.ts, pokemon-data/.
* Classes, interfaces, and type aliases: PascalCase, e.g. BattleScene, PokemonStats.
* Variables and functions: camelCase, e.g. playerLevel, calculateDamage().
* Constants: UPPER\_SNAKE\_CASE when they represent fixed global/configuration values, e.g. MAX\_PARTY\_SIZE.

Additional rules:

* Preserve TypeScript strictness. Prefer explicit, useful types over any.
* Keep modules focused and reasonably small. Avoid giant files and unnecessary abstractions.
* Use comments to explain non-obvious reasoning, constraints, or workarounds—not to narrate every line.
* Match import paths to the actual filename, including capitalization. Windows may tolerate casing mistakes that Linux hosting will not.
* Preserve existing imported asset filenames and folder capitalization by default. Restructure them when useful, but update every reference and verify the exact resulting paths. Do not repeatedly rename casing for cosmetic consistency.

## 6\. Static assets and paths

Static files are stored under public/. Vite serves files from that directory at the site root and copies them into the production build.

For example:

* File: public/assets/example.png
* Browser URL: /assets/example.png

Do **not** include /public in browser URLs.

Asset rules:

* Match the exact asset filename and capitalization.
* Do not create thousands of static imports or a hand-maintained lookup table when a predictable filename can be derived from a Pokémon's internal name.
* Keep asset-path construction in a small centralized helper when it is reused.
* Do not assume a local asset exists just because its path is in the code. Some asset collections are intentionally excluded from Git.

### Locally supplied, Git-ignored assets

The root .gitignore intentionally excludes these local collections:

* public/assets/audio/sfx/cries/

  * Cry filenames use the Pokémon internal name in uppercase with the .ogg extension.
* public/assets/graphics/pokemon/

  * Pokémon graphics are organized by variant (currently normal and shiny) and use the uppercase Pokémon internal name with the .png extension.
* public/assets/fonts/LXGWWenKai-Regular.ttf

  * Large CJK-capable font kept locally rather than committed.

These files remain available to the developer's local Vite project, but they are absent from a fresh Git clone and from builds made without the local files. Do not remove these ignore rules or add these assets to Git without the user's explicit direction.

### Future CDN support

The user expects that production may use remotely hosted assets delivered through a CDN. No CDN provider or production asset pipeline has been selected yet.

When implementing asset loading:

* Keep asset URLs centralized or configurable instead of scattering a hostname throughout the codebase.
* Preserve a convenient local-development asset source.
* Make it possible to configure a production asset base URL later.
* Do not select a provider, upload assets, or implement a deployment pipeline without being asked.
* Do not upload any asset to a public CDN until its redistribution rights have been checked.

## 7\. Imported resources, credits, and licensing

This repository contains resources imported from Pokémon Essentials and third-party add-ons, as well as separately sourced graphics such as type icons.

The public/credits/ directory contains attribution/source files. Preserve these files and update them when adding resources that require attribution.

Rules:

* Record the source, author, and applicable license or usage terms for imported resources.
* A code license does not automatically apply to the graphics, audio, fonts, or other media distributed alongside that code.
* Do not assume that an asset is redistributable just because it was included in an Essentials installation or add-on.
* If redistribution permission is unclear, keep the asset local and flag the uncertainty instead of publishing it or uploading it to a CDN.
* Do not delete or rewrite existing credits to make an asset appear unrestricted.
* Do not present AImon as being built on Pokémon Essentials. It is a separate Phaser project that uses selected resources and references.

## 8\. Localization

Avoid hardcoding player-facing text directly into game logic or UI components when it can reasonably be represented as localized data.

Keep internal identifiers (such as Pokémon internal names and type IDs) separate from translated display names. Asset filenames may follow their existing internal-name convention; translated labels should not require renaming those files.

Do not assume that the presence of an icon or font means every language variant is already available. The currently committed type-icon files are under public/assets/graphics/ui/types/english/.

## 9\. Git and change-management rules

Before editing:

1. Inspect the relevant files and current implementation.
2. Check git status and preserve the user's uncommitted work.
3. Read the relevant project instructions and nearby code before changing architecture.

While editing:

* Make focused changes that solve the requested task.
* Do not overwrite or revert unrelated user changes.
* Do not edit generated output in dist/ or installed dependencies in node\_modules/.
* Do not remove ignored local assets or alter .gitignore rules casually.
* Do not run destructive Git commands such as reset --hard, forced checkout, or mass deletion unless the user explicitly requests the operation and its consequences are clear.
* Do not commit, push, or modify the remote GitHub repository unless the user explicitly asks.

Before reporting completion:

1. Run npm run build when the change affects application code or configuration.
2. Run relevant manual checks when practical.
3. Report exactly what was changed and which checks actually passed.
4. Call out anything not tested, blocked by missing local assets, or dependent on an undecided design choice.

## 10\. How to work with the user

The user is coordinating development across multiple AI assistants. Be compatible with collaborative work:

* Treat the user's latest explicit instructions and decisions as authoritative.
* Inspect the repository rather than assuming another assistant's changes or an earlier conversation's file contents are still current.
* Explain important assumptions and tradeoffs briefly.
* When proposing a larger architectural choice, distinguish a recommendation from an established project decision.
* Prefer reversible, incremental work over large speculative rewrites.
* Do not claim to have run commands, viewed local files, tested the game, or changed the repository unless that action actually occurred.

