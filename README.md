# Recurrent Complex Volts

Recurrent Complex Volts brings classic Recurrent Complex structures and
worldgen to Minecraft 1.20.1 Forge. It loads old RC `.rcst` structure files,
generates bundled structures in newly generated chunks, and gives players an
in-game GUI for browsing, previewing, placing, exporting, editing, and diagnosing
structures.

The port is built around compatibility. Missing mod content degrades safely,
usually to air, instead of crashing a world. Old `.rcst` files remain the main
Recurrent Complex structure format.

Current public baseline: `0.7.0.0`.

Please consider supporting the developer at https://ko-fi.com/akiak

## Guide Contents

- [Requirements And Installation](#requirements-and-installation)
- [Updating And Existing Worlds](#updating-and-existing-worlds)
- [Permissions](#permissions)
- [Quick Start](#quick-start)
- [Main GUI Tabs](#main-gui-tabs)
- [Common GUI Workflows](#common-gui-workflows)
- [Worldgen And Frequency](#worldgen-and-frequency)
- [Structure Compatibility](#structure-compatibility)
- [Troubleshooting](#troubleshooting)
- [Runtime Notes And Known Limitations](#runtime-notes-and-known-limitations)
- [Building From Source](#building-from-source)

Detailed guides:

- [Structure Authoring](docs/user/structure-authoring.md)
- [Configuration And Troubleshooting](docs/user/configuration-and-troubleshooting.md)
- [Command Reference](docs/user/command-reference.md)

## Requirements And Installation

- Minecraft: `1.20.1`
- Forge: `47.4.10`
- Java: `17`
- Mod id: `reccomplex`

Install the matching `RecurrentComplexVolts-1.20.1-0.7.0.0.jar` in the instance's
`mods` folder. Multiplayer servers and connecting clients should all use the
matching jar. No content-library dependency is required beyond Minecraft and the
listed Forge version.

Do not use the 1.21.1 or 26.1.2 NeoForge jar on this target. A jar built for a
different Minecraft version or loader will not work.

## Updating And Existing Worlds

RC worldgen runs when chunks are first generated. Installing the mod in an
existing world works, but structures appear only in unexplored chunks; existing
chunks are not regenerated or retroactively populated.

Back up an important world before updating or importing a large structure pack.
Existing `config/reccomplex-common.toml` files retain their stored values when a
new version changes defaults. To adopt every current default, stop the game,
back up or rename that file, and let RC create a new one at next launch.

## Permissions

- Anyone can open the GUI and use read-only browsing, checks, audits, and most
  diagnostics.
- Permission level `2` is required for selection, preview, placement, confirm,
  cancel, undo, catalog reloads, predictive locating, and `/rc generate`.
- Permission level `4` is required for exports, editable copies, metadata and
  loot-generator/structure-list membership changes, missing-block replacement
  changes, and global balance edits.

In singleplayer, enable cheats when creating the world or temporarily use
`Open to LAN -> Allow Cheats`. On a dedicated server, use the normal operator
configuration.

## Quick Start

In game, run:

```text
/rc gui
```

Then use the GUI:

1. Open `Structs`.
2. Search or pick a structure.
3. Click `Preview` to show the ghost preview.
4. Click `Confirm` to place it, or `Cancel` to discard the preview.
5. Click `Undo` if a confirmed placement needs to be rolled back.

For built-in help and diagnostics:

```text
/rc help
/rc status
```

Optional unbound controls are available under
`Options -> Controls -> Key Binds -> Recurrent Complex` for opening the browser,
confirming or cancelling previews, and undoing the latest RC operation.

For the detailed command map, see the
[command reference](docs/user/command-reference.md).

## Main GUI Tabs

- `Structs`
  - Browse bundled and exported `.rcst` structures, inspect details, preview
    structures, confirm/cancel placements, undo, and pick random structures.
- `Schems`
  - Browse `.schem`, legacy `.schematic`, and Create/vanilla `.nbt` files,
    preview schematic imports, and convert schematics into editable `.rcst`
    structures.
- `Auth`
  - Work with selections, clipboard copy/paste, `.rcst` export, `.schem`
    export, marker painting, script marker authoring, weighted child pools, and
    loot/container authoring.
- `Balance`
  - Make RC worldgen rarer, commoner, disabled, or spaced out without opening
    the config file by hand.
- `Gen`
  - Edit one selected structure: make editable copies, set generation rules,
    tune natural/sapling/decoration/village metadata, edit supported
    transformers, repair missing blocks, and update basic metadata.

Most GUI buttons have hover tooltips. Text lines shortened with `...` also show
the full text on hover.

## Common GUI Workflows

### Try A Bundled Structure

1. Run `/rc gui`.
2. In `Structs`, select something like `SmallFortRuins`.
3. Click `Preview`.
4. If the ghost looks right, click `Confirm`.
5. Use `Undo` if you want to remove it again.

`Normal` placement is the recommended compatibility mode. `Raw` is a debug-style
direct copy mode and is not recommended for ordinary placement.

The preview ghost is advisory and sampled when a structure is large. Supported
transformers, terrain changes, scripts, block entities, entities, and final
safe-write checks run during confirmed placement. `/rc undo` covers the latest
undoable command-side edit while its in-memory history remains available; it
does not undo automatic worldgen or survive a restart.

### Generate A Structure With Its Natural Placer

`/rc generate` is the modern equivalent of old `/#gen`. Unlike ordinary manual
placement, it uses the structure's natural-generation placer and centered X/Z
anchor.

```text
/rc generate SmallFortRuins
/rc generate SmallFortRuins at 100 -200
/rc generate SmallFortRuins at 100 -200 y 62 rotate 1 mirror
```

For a player, the command shows a ghost first. `/rc confirm` queues the frozen
origin and transform through the safe deferred worldgen pipeline; `/rc cancel`
discards it. Without `at`, your current X/Z is used. `y` supplies the lower world
Y for this one request and bypasses surface selection.

Explicit generation intentionally ignores automatic selector gates such as
frequency, biome/dimension eligibility, spacing, bundled inclusion, allowlists,
and the global worldgen toggle. It still requires supported natural metadata and
keeps build-height, conformity, `ensureBlocks`, chunk-safety, script, Create, and
phased-placement checks.

### Find Or Identify RC Structures

Use vanilla locate for the aggregate nearest predicted RC natural structure:

```text
/locate structure reccomplex:natural_structure
```

Use RC's tick-sliced locator for a particular structure:

```text
/rc locate StonePlantMound
/rc locate StonePlantMound family
```

To identify a recorded RC footprint at your position:

```text
/rc here
/rc here at <x y z>
```

Footprint lookup is not block recognition. It does not backfill old worlds,
identify vanilla structures, or normally identify decoration trees and sapling
replacements because those footprint settings default to off.

### Import A Schematic

Put Sponge `.schem`, legacy MCEdit/Schematica `.schematic`, or
Create/vanilla structure-template `.nbt` files under:

```text
config/reccomplex/schematics
```

Create `.nbt` files may also stay in the instance-level `schematics/` folder
that Create uses.

Then:

1. Run `/rc gui`.
2. Open `Schems`.
3. Select the schematic.
4. Click `Preview`, then `Confirm` or `Cancel`.
5. Use `Convert` if you want to save it as an RC `.rcst` structure.

Schematic import uses the same compatibility resolver as `.rcst` structures
where possible, including old vanilla flattening, missing-block replacements,
and safe air fallback. Create/vanilla `.nbt` imports are marked as modern-origin
structures so block-entity and entity NBT can pass through generically when the
same modded block/entity types exist at placement time. `.nbt` imports are still
schematic sources; to use one in RC worldgen, convert it to `.rcst` first and
then configure its generation rules.

### Export A Custom Build

1. Run `/rc gui`, open `Auth -> Tools`, and click the tool buttons if you need
   the selector or inspector items.
2. Use `reccomplex:block_selector` on one corner to set `pos1`.
3. Crouch-use it on the opposite corner to set `pos2`.
4. Use `reccomplex:block_selector_floating` when you need to select an air
   corner.
5. Use `Auth -> Select` to inspect or adjust the inclusive cuboid.
6. Use `Auth -> Clip` to copy, paste-preview, export `.rcst`, or export `.schem`.

Modern in-game `.rcst` exports are marked for the same modern NBT passthrough as
Create/vanilla `.nbt` imports. Old unmarked `.rcst`, Sponge `.schem`, and legacy
`.schematic` imports continue to use the legacy-safe NBT import path.

Exported `.rcst` files are written under:

```text
config/reccomplex/structures/active/<id>.rcst
```

Files placed directly under `structures` or under `structures/active` are
loaded. Move files to `structures/inactive` to keep them installed but disabled.
Run `/rc reload` after manually adding or moving files, then click the GUI `R`
button if the open page needs refreshing.

### Make A Bundled Structure Editable

Bundled structures are read-only templates. To edit one:

1. Open `/rc gui`.
2. Select the bundled structure in `Structs`.
3. Open `Gen`.
4. Open `Meta`.
5. Use `Make edit` to create a config/user copy.
6. Edit the copied structure instead of the bundled original.

This protects bundled resources while still letting you customize them.

### Edit Generation, Rules, And Transformers

Open `/rc gui -> Gen` on an editable structure.

- `Nat`
  - Natural generation category, weight, baseline, dimension, and surface-style
    presets.
- `Trig`
  - Sapling and vanilla decoration replacement presets, weights, and trigger
    shifts.
- `Vil`
  - Bounded RC village-piece metadata such as biome preset, front direction,
    weight, cap, and shift.
- `Rules`
  - Simple biome and dimension allow/block rules. You can type values like
    `forest`, `plains`, `#minecraft:is_forest`, `overworld`, `nether`, `end`,
    or `all` without learning old RC expression syntax.
- `Xform`
  - Supported transformer presets and details for terrain blend, natural air,
    preserve air, and ruins.
- `Meta`
  - Make editable copies and edit authors, comment, weblink, and basic flags.
- `Fix`
  - Inspect missing blocks and choose replacement behavior.

GUI fields are explicit-apply fields. Typing a value does not change the
structure until you press the matching button.

### Choose Terrain Marker Meanings

`Auth -> Marks` paints placeholder blocks that tell RC how the destination
world should be treated during compatible placement:

| Marker | Placement meaning |
| --- | --- |
| `Neg space` | Preserve the block already present in the destination world. |
| `Nat air` | Clear the destination to air and participate in supported natural-air cleanup. |
| `Nat floor` | Retain existing solid terrain, or fill empty/replaceable space with dimension-appropriate natural terrain; it can also seed supported terrain blending. |

The modern authoring subset does not yet provide arbitrary liquid designation,
explicit top/filler-layer control, meaningful authoring for reserved
generic-space/generic-solid variants, or the full old generic-filling
table/editor. Imported metadata in the supported subset is still read;
unsupported reserved markers degrade conservatively by preserving the
destination block.

### Make A Custom Structure Generate Naturally

`/rc export` captures the structure only; it never opens a generation-settings
popup. After export, select the new id under `Structs`, open `Gen`, and configure
its generation behavior there.

1. Export the build or make an editable copy of a bundled structure.
2. Test it with `Check`, `Audit`, and a normal manual preview.
3. Open `Gen -> Nat`, select a category, then apply `Surf`, `Under`, or `Nether`.
4. Set its weight, dimension preset, biome/dimension rules, and allowed rotation
   or mirroring.
5. `Baseline` is the local saved Y level aligned to the sampled surface. It is
   not an absolute world height.
6. Click `Valid` and resolve any reported error.
7. Run `/rc generate <id>` to test the natural placer, then confirm or cancel.
8. Test automatic selection in newly generated chunks. Use
   `/rc worldgen inspect <id>` and `/rc worldgen rates <id>` when needed.

The current GUI does not expose every obscure legacy placer or raw expression.
Existing supported legacy metadata is still read, while common new structures
should use the conservative GUI presets and rule editor. See the
[structure authoring guide](docs/user/structure-authoring.md) for saplings,
decorations, villages, scripts, structure lists, mazes, markers, large
structures, and the exact supported boundaries.

### Understand Scripts, Structure Lists, And Mazes

- `Auth -> Script -> Spawn -> Set Child` creates a marker that directly names one
  child `.rcst`.
- `Auth -> Script -> Pool` creates or edits a named weighted child pool. A pool
  is not a separate file: each membership is saved as legacy `structureList`
  metadata inside that child `.rcst`.
- `Auth -> Script -> Spawn -> Set List` creates a marker that references an
  existing pool; it does not edit pool membership. At placement time RC chooses
  a member by weight and applies its saved shift and front.
- Bundled pool members are visible but read-only. Copy/export a child under an
  editable config id before adding it to a custom pool.
- Named pools honor each child's existing `dependencyExpression`. Unmatched or
  unsupported members stay visible but are excluded from weighted selection;
  set the child-wide condition under `Gen -> Expr -> Dep` or run
  `/rc gen expr dependency <childId> <expression>`.
- The old embedded per-marker table of weighted children remains deferred. The
  supported random mode is the named legacy structure-list system above.
- Existing supported `mazeGen` data is why bundled legacy mazes work. Designing
  a new maze graph, connector set, and room pool is not currently a supported
  GUI workflow.
- Child structures are useful for composition, but splitting a large structure
  into script children does not inherently make it place faster.

Simple child example:

1. Export or otherwise load the child `.rcst`, for example `MyRoom`.
2. Build the parent, open `/rc gui -> Auth -> Script -> Target`, and choose the
   marker position with the coordinate fields, `Here`, or `Look`. `Target` only
   moves the editor cursor; it does not place anything.
3. Open `Spawn`, enter the loaded child id, and click `Set Child`. This creates
   or updates an RC script marker at the selected target.
4. Adjust its shift, front, rotation, or mirror settings under `Xform` if needed.
5. Include the marker in the parent selection, export the parent, and test its
   placement.

The end-to-end child-pool tutorial is in
[Structure Authoring: Weighted Random Child Pools](docs/user/structure-authoring.md#weighted-random-child-pools).

### Fix Missing Blocks

Missing mod blocks become air by default. The `Gen -> Fix` page gives two
repair choices:

- `Set Global`
  - Saves a replacement rule in
    `config/reccomplex/missing-block-replacements.json`. This affects every
    structure with the same missing legacy block id.
- `Apply Here`
  - Rewrites only the selected editable `.rcst` structure so that one missing
    state becomes the typed modern blockstate. Other structures are unaffected.
- `Clear Global`
  - Removes the selected global replacement rule.

Examples of valid replacements:

```text
minecraft:calcite
minecraft:oak_log[axis=y]
```

Bundled structures must be copied with `Meta -> Make edit` before `Apply Here`
can rewrite them.

### Make Structures Rarer Or Commoner

Open `/rc gui -> Balance`.

- `0` disables a multiplier-based path.
- `0.5` means half as common.
- `1` means normal.
- `2` means twice as common.
- `rarity` is the old denominator: higher means rarer.
- `Bundled on/off` controls whether bundled mod structures participate in
  default RC worldgen catalogs and whether they appear in `/rc gui -> Structs`.
  It does not delete bundled structures or block explicit command/manual use,
  copying, checks, or audits. Explicit `worldgen.allowedStructureIds` entries
  can still opt bundled natural structures back in.

`Balance` covers natural structures, category multipliers, saplings,
decorations, villages, and optional spacing/anti-clumping for RC natural
structures.

### Add Loot To Containers

Open `/rc gui -> Auth -> Loot`.

- `Container`
  - Pick a chest/barrel/container with `Here`, `Look`, or coordinates.
  - Use `Box` to assign an RC inventory generator marker (`RCIG`).
  - Use `Van` to assign a normal vanilla/datapack loot table.
  - Use `Clear` to remove RC/vanilla loot data from that container.
- `Generator`
  - Use `Gen` to create/delete simple `.rcig` inventory generators and set
    their min/max item-count range.
  - Use `Entry` to add/update/remove weighted item entries.
  - Use the small arrows beside generator fields to pick from discovered
    `.rcig` files instead of remembering names by hand.

Vanilla loot tables are still supported. RC `.rcig` generators are useful when
you want old-RC-style, in-game editable weighted loot that can also be preserved
inside exported structures.

User `.rcig` files can live directly under
`config/reccomplex/structures/active/<id>.rcig`. The older
`active/inventory/<id>.rcig` layout is still read for compatibility. Loose
`.rcig` files directly under `config/reccomplex/structures` are also loaded;
files under `structures/inactive` are ignored. Vanilla
loot tables use normal datapack or mod resource locations, for example
`data/<namespace>/loot_tables/chests/my_loot.json`, then type
`<namespace>:chests/my_loot` in the GUI.

## What The Mod Adds

- Classic RC `.rcst` structure loading for Minecraft 1.20.1 Forge.
- Natural Overworld and Nether structures through a deferred compatibility
  worldgen bridge.
- Bundled ruins, trees, dungeons, mazes, villages, Nether structures,
  decorations, sapling replacements, terrain blending, and script-generation
  subsets.
- GUI-first structure browsing, ghost preview, confirm/cancel, undo, schematic
  import/export, authoring, generation editing, transformer editing, script
  marker editing, loot/container editing, missing-block repair, and worldgen
  balance controls.
- Missing-content tolerance for old structure packs from modded 1.12.2 worlds.
- Compatibility audits and diagnostics for large imported structure libraries.
- Vanilla locate support for aggregate RC natural structures:

```text
/locate structure reccomplex:natural_structure
```

Specific predictive RC locating is also available:

```text
/rc locate StonePlantMound
/rc locate StonePlantMound family
```

## Essential Commands

The GUI is the normal starting point. These commands are the useful backup and
diagnostic surface:

- `/rc gui`
  - Open the browser/editor GUI.
- `/rc help`
  - Show in-game help topics.
- `/rc status`
  - Show version, structure count, user folders, and major generation toggles.
- `/rc reload`
  - Reload structure and generation catalogs after manual file changes.
- `/rc generate <id>`
  - Preview explicit natural-style generation at your current X/Z, then use
    `/rc confirm` or `/rc cancel`.
- `/rc here`
  - Identify recorded RC structure footprints at your current position.
- `/rc locate <id> [family]`
  - Start a tick-sliced predictive search for one natural RC structure id.
- `/rc worldgen balance`
  - Show the current frequency/balance settings and config path.
- `/rc worldgen status`
  - Show worldgen bridge, queue, spacing, and diagnostics state.
- `/rc missing dump [filter]`
  - Write a detailed missing-block report for old structure packs under
    `config/reccomplex/reports`.
- `/rc loot list`
  - List RC inventory generators for container loot authoring.
- `/rc check <id>`
  - Inspect one structure without placing it.
- `/rc audit one <id>`
  - Audit one structure for compatibility issues.

For full syntax, see the
[command reference](docs/user/command-reference.md).

## Authoring Tool Items

The current GUI-backed authoring workflow has three item tools:

- `reccomplex:block_selector`
  - Use on a block to set `pos1`.
  - Crouch-use on a block to set `pos2`.
- `reccomplex:block_selector_floating`
  - Selects along your look vector, including air endpoints.
  - Normal use sets `pos1`; crouch-use sets `pos2`.
  - Ctrl-scroll while holding the item adjusts range from `0` to `40` blocks.
- `reccomplex:inspector`
  - Read-only block/state/block-entity inspection.
  - Reports RC internal placeholders and script/action markers.

Useful gives:

You can also use `/rc gui -> Auth -> Tools` to get these three items from
icon buttons.

```text
/give @p reccomplex:block_selector
/give @p reccomplex:block_selector_floating
/give @p reccomplex:inspector
```

## Structure Compatibility

Old 1.12.2 Recurrent Complex `.rcst` files are treated as user data. Imported
structures and structures created with `/rc export` use the same `.rcst` format;
old data is adapted at load, preview, placement, export, and generation time.

Compatibility policy:

- preserve namespaced ids where possible;
- convert common old metadata/blockstate forms to modern 1.20.1 blockstates;
- replace unknown or missing blocks with air unless a replacement is configured;
- skip unknown or missing items/entities safely;
- preserve block entity NBT only where the modern target is known and safe;
- report conversion gaps through GUI pages, commands, reports, and logs.

Missing block replacement rules are global unless you use `Apply Here` on an
editable structure.

## Worldgen And Frequency

Forge writes the common config relative to the active game or server instance:

```text
config/reccomplex-common.toml
```

The GUI `Balance` tab is the easiest way to change the major knobs. Commands and
manual config edits are still available for pack maintainers.

For ordinary overall tuning, leave `rarity` at `1` and change the natural
multiplier first. `rarity` remains as the older extra denominator for
compatibility. Individual `.rcst` weights control competition between eligible
structures and are not direct spawn percentages.

Important groups:

- `worldgen`
  - Natural generation, deferred placement limits, optional spacing, native
    locate support, bundled-default inclusion, complement behavior, active
    placement fast-lane limits, and diagnostics.
- `saplings`
  - RC sapling replacement.
- `decorations`
  - Tree, mushroom, cactus, desert well, and fossil replacement. Tree
    decoration first uses `treeChunkChance` (`0.10` by default), then the
    conservative `treeBaseWeight`, and accepts at most one RC tree replacement
    attempt per chunk to avoid deferred-placement pressure in dense forests.
- `villages`
  - Bounded RC village-piece bridge. `attemptChance` softly controls how often
    eligible village chunks try RC pieces, `baseWeight` controls how strongly
    RC pieces compete when attempted, and `budget` remains a safety cap for
    accepted RC cost in one detected village.

Older config section names are migrated automatically where possible.

Existing generated configs do not replace stored values when a later release
changes a default. Balance GUI changes apply live and save the file. When editing
TOML manually, stop the game or server first. `/rc reload` refreshes structure
and generation catalogs; it is not a general TOML reload command.

See [Configuration And Troubleshooting](docs/user/configuration-and-troubleshooting.md)
for current defaults, spacing, terrain roughness, sapling/decoration/village
tuning, fast-lane behavior, and diagnostic procedures.

## Troubleshooting

### No Structures Appear

- Explore newly generated chunks, not terrain that already existed before the
  mod or config change.
- Run `/rc worldgen status`, `catalog`, `balance`, and `recent`.
- Check `worldgen.enabled`, bundled inclusion, `allowedStructureIds`, spawn
  distance, rarity, and the natural multiplier.
- Remember that an older generated config retains its stored values.

### One Structure Does Not Generate

Run `/rc check <id>`, `/rc gen validate <id>`,
`/rc worldgen inspect <id>`, and `/rc worldgen rates <id>`. Use
`/rc generate <id>` to test its natural placer separately from automatic
selection gates.

### A Structure Appears Slowly Or In Pieces

Run `/rc worldgen queue` while its chunks are loaded. Loaded active placements
and loaded ledger complement slices should report fast-lane activity. Work still
pauses for unloaded chunks or failed safe-write checks, and truly enormous
structures are not instant regional jobs.

If ordinary chunks stop loading too, capture `logs/latest.log` and several queue
and recent outputs before leaving the world. The log from the stalled session is
more useful than a later clean restart.

### `/rc here` Finds Nothing

The command uses stored footprints. It cannot backfill older placements, detect
vanilla structures, or normally identify decoration/sapling structures while
their `recordFootprints` settings are disabled.

### Imported Blocks Become Air

This is the intentional missing-mod fallback. Use `Gen -> Fix`,
`/rc missing one <id>`, or `/rc missing dump` to inspect and replace missing
legacy ids.

The full symptom-by-symptom checklist is in
[Configuration And Troubleshooting](docs/user/configuration-and-troubleshooting.md).

## Runtime Notes And Known Limitations

Worldgen uses a deferred compatibility bridge. RC structures are selected during
worldgen, then placed later on the server thread when affected chunks are safe
to touch. This is intentional; it avoids unsafe chunk-worker world mutation while
letting old multi-chunk structures work in modern Minecraft.

Already-started loaded natural, village, and script-child placements get a
bounded active-placement boost so visible structures finish sooner. Loaded
chunk-complement slices for ledgered natural structures get the same kind of
bounded boost. The knobs live under `worldgen.activePlacement` and
`worldgen.chunkComplements`; decorations remain on the conservative pacing path.

Known limitations:

- the current GUI is a modern browser/editor front-end, not the full old 1.12.2
  editor stack;
- full old transformer table parity, exact expression parity, and some advanced
  legacy editor systems are still deferred;
- village pieces use the bounded compatibility bridge, not true modern jigsaw
  pool injection;
- script authoring covers command markers, simple child structure spawners, and
  creation/editing/use of named RC weighted structure lists, while `mazeGen`,
  holder scripts, custom environment matchers, and embedded per-marker
  child-table editing remain deferred;
- modern-origin `.nbt` imports and in-game `.rcst` exports preserve unknown
  modded block-entity/entity NBT generically, but rotation and mirroring only
  transform block states, positions, and supported entity orientation. Unknown
  mod-specific machine internals are not interpreted and may need in-game
  reconfiguration after transformed placement;
- loot authoring covers vanilla loot-table assignment and practical `.rcig`
  inventory generators; full old RC book/artifact generator table editing is
  still deferred;
- missing mod content degrades safely, usually to air, rather than being
  automatically remapped to unrelated vanilla blocks;
- already-started active phased decoration placements can still be pruned if
  their chunks stay unloaded too long under very fast creative flight pressure;
- parked root decorations may be evicted under extreme planned-store pressure
  so accepted village and natural structures can still park and resume;
- true regional megastructure jobs are still future work, so enormous imported
  builds may need authoring strategy or later chunk-sliced placement support.

When reporting bugs, include `logs/latest.log`, any crash report, the world
seed/dimension/position if relevant, the GUI page or `/rc` command used, the
structure/schematic id, and a screenshot for visible placement or GUI issues.

## Building From Source

Most users do not need this section. It is for developers or pack maintainers
building the jar locally.

Use a Java 17 JDK and the included Gradle wrapper:

```sh
./gradlew build
```

Set `JAVA_HOME` through your operating system or development environment if Java
17 is not already the active JDK.

The built mod jar is written under `build/libs/`.

For a development client:

```sh
./gradlew runClient
```

For a development server:

```sh
./gradlew runServer
```

The modern port compiles from `src/modern/java` and `src/modernTest/java`. The
legacy 1.12.2 source remains in `src/main/java` as reference material and is not
compiled by the modern source set.

Build target details:

- Minecraft: `1.20.1`
- Forge: `47.4.10`
- Java: `17`
- Gradle wrapper: `8.8`

For release testing, use `./gradlew verifyReleaseJar` plus the checklist in
[`docs/porting/release-checklist.md`](docs/porting/release-checklist.md).
