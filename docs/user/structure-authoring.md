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

`/rc export` writes the selected blocks and NBT directly to an RC `.rcst`
structure. There is no separate old or new RC export mode. The archive contains
the `structure.json` and `worldData.nbt` used by RC structures.

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

Export captures the structure; it does not open the old 1.12 generation dialog.
After export, select the new id under `Structs`, open `Gen`, and configure its
natural, sapling, decoration, village, rules, transformers, flags, and metadata
there. Test manual placement before enabling a generation preset.

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

`Auth -> Marks` paints the currently supported RC placeholder meanings:

- `Neg space` preserves the block already in the destination world.
- `Nat air` clears the destination to air and participates in the supported
  natural-air cleanup transformer.
- `Nat floor` asks the compatible placer for natural terrain fill and can seed
  the supported terrain-blend transformer.
- Smart marking derives the supported floor/air/space markers from an ordinary
  build selection.

These are real RC authoring blocks written by the GUI action (or `/rc mark ...`),
not invisible modes attached to normal air. There are no separate modern GUI
buttons for every old generic fill designation such as arbitrary liquid or
top-layer tables; imported supported marker/transformer metadata is still read,
but that broader old filling editor remains outside this authoring subset.
Marker painting is an immediate command-side edit and can be undone while the
undo entry remains in memory.

## Scripts And Child Structures

`Auth -> Script` supports:

- weighted command markers;
- one explicit child-structure spawner;
- authoring and using a named weighted RC structure list;
- child shift, front, rotation, and mirror settings;
- spawn and redstone trigger toggles.

There is no separate “script block” to take from the creative inventory. First
choose a world position in `Auth -> Script -> Target`; that only moves the
editor's cursor. Then an action such as `Set One`, `Set Child`, or `Set List`
places or updates the RC script marker block and its block-entity data at that
target. Include that marker position inside the parent selection before
exporting the parent. During normal compatibility placement, RC interprets the
marker and does not leave it as an ordinary decorative block.

The three child modes are different:

- **Simple child:** `Spawn -> Set Child` creates a marker that directly names one
  child structure.
- **Named random list:** `Pool` edits weighted membership, while
  `Spawn -> Set List` creates a marker that references that list. This is the
  supported random-child workflow.
- **Embedded per-marker table:** the old advanced table that stored several
  weighted child choices directly on one marker is still deferred. Do not
  confuse it with a named structure list.

## Weighted Random Child Pools

A named pool is deliberately distributed across its children. Every member is a
legacy `structureList` entry inside that child's `.rcst`, storing `id`, `listID`,
`weight`, `positionX/Y/Z`, and `front`. There is no pool/list file to export.
Adding the first member creates the discoverable list; removing its last member
makes the list disappear.

Complete GUI workflow:

1. Build and export each child, for example `MyRoomA` and `MyRoomB`. Test both as
   normal structures first.
2. Open `Auth -> Script -> Pool`.
3. Type a new list id such as `my_rooms`, type `MyRoomA`, choose its positive
   weight, X/Y/Z shift, and horizontal front, then click `Add`.
4. Keep `my_rooms`, type `MyRoomB`, choose its values, and click `Add`. Use the
   list and member arrows to browse; `Update` and `Remove` act on the displayed
   membership.
5. Build the parent. Open `Auth -> Script -> Target` and choose the block where
   the child origin should be invoked.
6. Open `Spawn`, pick `my_rooms` with the discovered-list arrows (or type it),
   then click `Set List`. This creates the marker reference; it does not create
   or modify the pool.
7. Select the complete parent including the marker, export it, and test repeated
   normal placements. Only pool members should be chosen, in proportion to
   their weights. Verify each shift, front, parent rotation, and mirroring.

Bundled memberships appear in the Pool browser for compatibility but are
read-only. Use `/rc copy <sourceId> <newId>` or export/capture an editable child,
then add that editable id to the custom pool. Imported duplicate entries are
shown separately using stable internal entry keys; the GUI can select one
specific duplicate, while pair-based commands reject the ambiguous mutation.

Command equivalent:

```text
/rc struclist add my_rooms MyRoomA
/rc struclist weight my_rooms MyRoomA 3
/rc struclist shift my_rooms MyRoomA 1 0 -2
/rc struclist front my_rooms MyRoomA east
/rc struclist add my_rooms MyRoomB
/rc struclist show my_rooms
/rc script struc list set <x y z> my_rooms
```

### Conditional Pool Members

A pool member can use the child structure's existing top-level dependency
expression. For example, make `QuarkRoom` eligible only when Quark is installed:

```text
/rc gen expr dependency QuarkRoom mod:quark
/rc struclist show my_rooms
```

The member remains visible in `Auth -> Script -> Pool` and `/rc struclist show`,
but it is marked excluded and receives no share of the random selection when
the expression is unmatched or unsupported. The remaining eligible weights are
used normally. If every member is excluded, the marker safely places no child
and reports which child dependencies prevented selection.

Dependencies belong to the child structure, not to one membership. They apply
everywhere that child participates in named pools. To make the same build
conditional in only one pool, copy/export it under another id, add that copy to
the pool, and assign the dependency to the copy. Use
`Structs -> Gen -> Expr -> Dep` or
`/rc gen expr dependency <childId> <expression>` to edit it. Clearing the
expression restores eligibility immediately without `/rc reload`.

Explicit/manual placement and direct `Set Child` markers remain available when
a dependency is unmatched, matching the legacy distinction between explicit
structure lookup and active named-list membership. Missing content still uses
the normal safe air fallback during those explicit placements.

List edits take effect immediately; `/rc reload` is not required. Manual file
changes still require the normal reload.

The runtime can expand the supported bounded subset of existing `mazeGen` and
`strucGen` data, which is why bundled legacy mazes work. There is not yet a
supported GUI or command workflow for designing/exporting a new maze graph,
connectors, and room pool from scratch, so there is no maze-export option to
find. Existing maze metadata is runtime compatibility, not a completed modern
maze authoring tool. Do not choose maze mode merely to split a large building.

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
