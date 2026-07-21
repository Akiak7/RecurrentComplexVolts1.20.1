# Structure Authoring Guide

This guide covers the supported in-game workflow for creating and configuring
Recurrent Complex Volts structures. The GUI is the normal editor. Directly
editing `structure.json` is an advanced compatibility option, not a requirement
for the workflows below.

For exact command syntax, see the [command reference](command-reference.md).

## Before You Start

- Authoring and placement require cheats or operator permissions.
- Permission level `2` covers selection, preview, placement, confirmation, and
  undo. Permission level `4` covers exports and file or metadata changes.
- Back up an important world before doing large edits. `/rc undo` is an
  in-memory, per-player command history, not a replacement for a world backup.
- The in-game selection/export limit is `262144` blocks of cuboid volume,
  including air.

## Structure, Schematic, And Source Types

- `.rcst` is the primary RC structure format. It contains `structure.json` for
  metadata and `worldData.nbt` for saved world contents.
- `.schem`, legacy `.schematic`, and Create/vanilla `.nbt` files are interchange
  sources. Convert them to `.rcst` before configuring RC worldgen.
- Bundled structures are read-only templates inside the mod jar.
- User structures are editable files under
  `config/reccomplex/structures/active`.
- A modern export or converted modern `.nbt` is marked for generic modern
  block-entity and entity NBT passthrough. Old files continue to use the safer
  legacy conversion path.

## Capture A Build

1. Build the structure in a safe authoring world.
2. Run `/rc gui`, open `Auth -> Tools`, and take the selector items.
3. Use `reccomplex:block_selector` on one corner for `pos1`.
4. Crouch-use it on the opposite corner for `pos2`.
5. Use `reccomplex:block_selector_floating` when an endpoint is in air. Hold
   Ctrl and scroll to change its range.
6. Check the inclusive dimensions and volume under `Auth -> Select`.
7. Add any RC markers, scripts, or container loot before export.
8. Open `Auth -> Clip`, enter an id, and export `.rcst`.

The lower selection corner becomes local `0 0 0`. Saved air inside the selected
cuboid is part of the structure data, although compatibility transformers and
markers can change what it does during placement.

Export ids may contain letters, numbers, `.`, `_`, `-`, and `/`. A slash creates
a nested id such as `my_pack/fort`. Blank path segments, `.` and `..` are
rejected.

The default output is:

```text
config/reccomplex/structures/active/<id>.rcst
```

Files in the older `active/structures` layout remain supported. Files under
`structures/inactive` stay installed but are not loaded. Use `/rc reload` after
adding or moving files outside the game, then use the GUI `R` button to refresh
an open browser page.

## Test Manual Placement First

Before enabling worldgen:

1. Run `/rc check <id>` and `/rc audit one <id>`.
2. Preview from `Structs`, or run `/rc preview place <id>`.
3. Test rotation and mirroring if they will be enabled.
4. Confirm in a disposable area and inspect blocks, block entities, entities,
   scripts, doors, and terrain interaction.
5. Use `/rc undo` immediately if the confirmed command placement is wrong.

Use `Normal` compatibility placement for real testing. `Raw` directly copies
saved blocks and is intended for diagnosing conversion or transformer behavior.
The ghost is advisory and sampled for large structures; final compatibility and
safe-write checks happen when placement runs.

## Make A Bundled Structure Editable

1. Select it in `/rc gui -> Structs`.
2. Open `Gen -> Meta`.
3. Enter a new id and click `Make edit`.
4. Edit the new config copy.

The command equivalent is `/rc copy <sourceId> <newId>`. The bundled original is
never modified.

## Enable Natural Worldgen

Start with an exported structure or editable copy:

1. Select it in `Structs`, then open `Gen -> Nat`.
2. Choose a category: `decoration`, `frequent`, `adventure`, `rare`, or
   `ultrarare`.
3. Apply `Surf`, `Under`, or `Nether` to create a conservative supported natural
   preset.
4. Set `Weight` if this structure should compete more or less strongly with
   peers in its category.
5. Set the dimension preset and open `Rules` for biome or dimension allow/block
   rules.
6. Set `Rot` and `Mir` according to whether every orientation is valid.
7. Click `Valid` and resolve any reported problem.
8. Test the natural placer with `/rc generate <id>`, inspect the preview, then
   run `/rc confirm` or `/rc cancel`.
9. Explore newly generated chunks for automatic-worldgen testing. Existing
   chunks are not retroactively regenerated.

`Baseline` is the local Y level inside the saved structure that should meet the
sampled surface. Increasing it moves more of the structure below that surface.
It is not an absolute world Y coordinate.

`/rc generate <id> y <y>` supplies a one-time lower world Y for that explicit
generation request. The current GUI preset editor does not expose a general
absolute-Y natural-worldgen preset. Existing legacy files with supported inline
or `generationY` placer data can still be read, but advanced placer construction
is outside the current GUI authoring subset.

Natural weight is only one part of selection. Category chance, global frequency,
biome and dimension rules, roughness, liquid/conformity checks, spawn distance,
optional spacing, and safe deferred placement can all affect the final result.
Use these when diagnosing a structure:

```text
/rc gen show <id>
/rc gen validate <id>
/rc worldgen inspect <id>
/rc worldgen rates <id>
```

## Saplings, Decorations, And Villages

`Gen -> Trig` provides sapling and vanilla-decoration replacement presets.

- For saplings, choose the sapling type and single or `2x2` pattern, apply the
  preset, set its weight, and adjust trigger shift if the saved trunk is not
  aligned with the sapling origin.
- For decorations, choose tree, big mushroom, cactus, desert well, or fossil,
  apply the preset, set its weight and shift, then use `Rules` for climate and
  dimension targeting.
- These paths compete with vanilla triggers; their weights are not the same as
  natural-structure category frequency.

`Gen -> Vil` configures the bounded village compatibility bridge:

- `Front` describes the piece's intended road-facing side.
- `Shift` aligns the saved piece with its chosen village anchor.
- `Weight` controls competition among eligible RC village pieces.
- `Cap` limits this individual piece per detected village.
- The biome preset limits village styles in which it can participate.

The bridge places pieces near detected modern village pieces. It does not add RC
pieces to vanilla jigsaw pools, so it cannot provide exact jigsaw road
negotiation.

Preset buttons create or replace supported generation metadata. Make an editable
copy before experimenting with a complex legacy structure that has several
special generation entries.

## Terrain And Marker Tools

`Gen -> Xform` exposes the supported transformer subset:

- `Blend` shapes surrounding natural terrain toward the structure.
- `Air` handles natural-air and preserve-air behavior.
- `Ruins` applies the supported decay, gravity, vines, and cobweb behavior.

`Auth -> Marks` paints RC placeholder blocks such as negative space, natural air,
natural floor, and barriers. Smart floor/space tools help prepare ordinary
builds for terrain-aware placement. Marker painting is an immediate command-side
edit and can be undone while the undo entry remains in memory.

## Scripts And Child Structures

`Auth -> Script` supports:

- weighted command markers;
- one explicit child-structure spawner;
- an existing RC structure-list id;
- child shift, front, rotation, and mirror settings;
- spawn and redstone trigger toggles.

A structure list is a named pool supplied by legacy `structureList` generation
entries. A script marker in list mode asks RC to choose from that existing
weighted pool. The current editor can point at a discovered list id, but it does
not create or edit the list's weighted entries in game.

The runtime can expand the supported bounded subset of existing `mazeGen` and
`strucGen` data, which is why bundled legacy mazes work. There is not yet a
supported GUI workflow for designing a new maze graph, connectors, and room
pool from scratch. Do not choose maze mode merely to split a large building.

Child structures are useful for reusable or optional sections. They are queued
as separate safe placements and do not guarantee that a very large build will
appear instantly.

## Container Loot

Use `Auth -> Loot` after choosing a container target:

- `Van` assigns a normal vanilla/datapack loot table.
- `Box` assigns an RC `.rcig` inventory generator marker.
- `Gen` creates or edits a simple weighted `.rcig` generator.
- `Clear` removes RC and vanilla loot metadata from the target container.

User `.rcig` files normally live beside structures under
`config/reccomplex/structures/active`. Datapack loot tables continue to use
normal namespaced ids such as `my_pack:chests/fort`.

## Missing Mod Content

Unknown legacy blocks resolve to air instead of crashing placement.

1. Open `Gen -> Fix` on the affected structure.
2. Use `Set Global` when every structure should use the same replacement.
3. Use `Apply Here` to rewrite only this editable `.rcst`.
4. Use `/rc missing dump` for a report across a large imported library.

Global rules live in
`config/reccomplex/missing-block-replacements.json`. A local apply preserves the
rest of the archive and rewrites only the selected saved block mapping.

## Large Structures

The in-game selector cannot export a cuboid larger than `262144` positions.
Externally supplied legacy or schematic files can be larger, but extremely large
single structures remain demanding.

Natural structures use deferred safe placement. Eligible plain natural
structures can use the chunk-complement ledger, allowing loaded chunks to finish
their own slices without force-loading the full footprint. Loaded active work
and loaded complement slices receive bounded fast-lane boosts. Unloaded or
unsafe chunks still wait.

This is not an instant regional megastructure system. A build hundreds of blocks
wide may still complete over time as its chunks load. Splitting into children is
an authoring choice, not a required optimization, and script-child pacing can be
slower than one eligible plain ledgered structure.

## Final Checklist

- The structure passes `Check`, `Audit`, and `Valid`.
- Rotation and mirror settings are safe for every enabled transform.
- The local baseline and trigger shift are aligned correctly.
- Biome and dimension rules express the intended environments.
- Missing blocks have intentional air fallback or replacement rules.
- Containers, block entities, entities, and scripts survive a normal preview and
  confirmed placement.
- Natural generation is tested in newly generated chunks with clean or known
  config values.
- The `.rcst` and world are backed up before large metadata experiments.

See also:

- [Main README](../../README.md)
- [Command Reference](command-reference.md)
- [Configuration And Troubleshooting](configuration-and-troubleshooting.md)
