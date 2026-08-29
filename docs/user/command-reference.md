# Command Reference

This is the compact user-facing command map for the current Recurrent Complex
Volts build. In game, start with `/rc help` and `/rc status`.

## Permissions

- Read-only browsing, checks, audits, and most diagnostics are available without
  elevated permissions.
- Permission level `2` is required for selection editing, previews, placement,
  confirmation, undo, catalog reloads, predictive locating, and `/rc generate`.
- Permission level `4` is required for file, metadata, loot-generator, missing
  replacement, and global balance changes.
- In singleplayer, enable cheats when creating the world or temporarily use
  `Open to LAN -> Allow Cheats`. On a dedicated server, use the normal operator
  configuration.

## Basics

- `/rc help` - list help topics.
- `/rc help <topic>` - show `basics`, `library`, `place`, `authoring`,
  `worldgen`, `diagnostics`, or `config`.
- `/rc status` - show version, structure count, user structure folder, and
  major generation toggles.
- `/rc gui` or `/rc gui browser` - open the modern structure/schematic browser,
  preview launcher, exported-structure generation/rules/transformer panel,
  and Auth tab.
- Controls -> Key Binds -> `Recurrent Complex` exposes unbound keys for opening
  the RC browser, confirming/canceling a pending preview, and undoing the latest
  RC operation.
- `/rc test` - confirm the command bridge is loaded.
- `/rc reload` - reload structure and generation catalogs after file changes.

## Library And Compatibility

- `/rc list [filter]` - list the first page of bundled and exported structure
  ids.
- `/rc list page <page>` - browse all structures by page.
- `/rc list filter <text> page <page>` - search and browse by page.
- `/rc info <id>` - show dimensions, metadata, generation/script notes,
  compatibility warning count, and next commands.
- `/rc copy <sourceId> <newId>` - copy any loadable bundled/user structure into
  the editable config structure folder under a new id.
- `/rc random [filter]` - pick a random matching structure id without placing
  it.
- `/rc last` - show the last id picked by `/rc info` or `/rc random`.
- `/rc gui` - browse structures and schematics with search, paging, details,
  transform controls, preview, confirm/cancel, undo, schematic conversion,
  a Meta page for editable copies and basic authors/comment/weblink fields,
  generation/rules/transformer/flag editing for exported config
  structures, and an Auth tab for selection/clipboard/export/marker/script/loot
  workflows.
- `/rc schem list [filter]` - list `.schem`, `.schematic`, and
  Create/vanilla `.nbt` schematic files under `config/reccomplex/schematics`
  and the instance-level `schematics/` folder.
- `/rc schem preview <name> [at <x y z>] [rotate <0-3>] [mirror]` - preview a
  schematic import through the same confirm/cancel/undo flow as structures.
  Modern-origin `.nbt` previews preserve block-entity/entity NBT generically
  when the target block/entity type exists.
- `/rc schem convert <name> <id>` - convert a schematic into a persistent
  `.rcst` structure. `.nbt` files are import sources; convert them before
  configuring RC worldgen. The modern-origin marker is preserved by conversion.
- `/rc check <id>` - inspect structure size, markers, scripts, and conversion
  report without placing. The output includes whether legacy-safe or modern NBT
  passthrough placement will be used.
- `/rc audit` - audit all loadable structures.
- `/rc audit one <id>` - audit one structure.
- `/rc audit filter <text>` - audit matching structures.
- `/rc missing list` - list global missing-block replacement rules from
  `config/reccomplex/missing-block-replacements.json`.
- `/rc missing one <id>` - scan one structure and list missing legacy block ids,
  metadata, counts, and current replacements.
- `/rc missing filter <text>` - scan matching structures and summarize top
  missing block ids.
- `/rc missing dump [filter]` - scan all or matching structures and write a
  full missing-block report under `config/reccomplex/reports`. This is the
  detailed bulk-audit path for large old structure packs; normal worldgen logs
  intentionally summarize expected missing-mod fallbacks instead of warning for
  every block id.
- `/rc missing set <legacyId[@metadata]> <modernBlockState>` - replace a
  missing block globally instead of falling back to air. Examples:
  `quark:limestone minecraft:calcite` or
  `quark:limestone@3 minecraft:oak_log[axis=y]`.
- `/rc missing apply <structureId> <legacyId[@metadata]> <modernBlockState>` -
  permanently repair only that editable config structure. Bundled/read-only
  structures must be copied first.
- `/rc missing clear <legacyId[@metadata]>` - remove one replacement rule.
- `/rc missing reload` - reload the replacement JSON file after manual edits.

Missing block replacement rules are non-destructive. They affect check/audit,
preview ghosts, placement, worldgen, and other paths that use the normal legacy
block resolver, but they do not rewrite bundled or user `.rcst` files. Use
`/rc missing apply` or GUI `Gen -> Fix -> Apply Here` when you want to save a
repair into one editable `.rcst` file instead of using a global rule.

## Placement

- `/rc preview place <id>` - show placement bounds near the player without
  mutating the world.
- `/rc preview place <id> at <x y z> [rotate <0-3>] [mirror]` - preview an
  exact transformed origin.
- `/rc confirm` - commit the pending previewed placement or paste.
- `/rc cancel` - clear the pending preview without changing blocks.
- `/rc undo` - restore the latest undoable command-side world edit in the
  current dimension.
- `/rc undo show` - show the latest undo entry and stack count.
- `/rc undo clear` - clear your in-memory undo history.
- `/rc place <id>` - place in compatibility mode near the player.
- `/rc place <id> raw` - debug direct-copy placement.
- `/rc place <id> rotate <0-3> [mirror]` - place transformed.
- `/rc place <id> at <x y z> [raw|mirror|rotate <0-3>]` - place at an exact
  origin.

## Authoring

- `/rc select pos1 [x y z]` and `/rc select pos2 [x y z]` - set the inclusive
  cuboid corners.
- `/rc select look pos1 [range]` and `/rc select look pos2 [range]` - select
  along your look ray, including air endpoints when no block is hit.
- `/rc select show`, `clear`, `expand`, `shrink`, `move`, and
  `resize <face> <amount>` - inspect and edit the selection. Resize grows one
  named face outward with positive values and shrinks it inward with negatives.
- `/rc copy` with no arguments and `/rc paste` - temporary in-memory selection
  copy/paste.
- `/rc preview paste` - preview a clipboard paste before committing it with
  `/rc confirm`.
- `/rc mark negative_space`, `natural_air`, `natural_floor`, `barrier`,
  `smart_floor`, `smart_space`, and `smart` - paint RC authoring markers.
- `/rc script show <x y z>` - inspect one RC command or structure-spawner
  marker.
- `/rc script command set <x y z> <command>` - create or replace an undoable
  weighted command marker at an air/existing-script position.
- `/rc script command list <x y z>` - print the current weighted command entries.
- `/rc script command add <x y z> <weight> <command>` - append a weighted
  command entry to a command marker.
- `/rc script command update <x y z> <index> <weight> <command>` - replace one
  1-based weighted command entry.
- `/rc script command remove <x y z> <index>` - remove one 1-based weighted
  command entry.
- `/rc script command clear <x y z>` - clear command entries while keeping the
  command marker block.
- `/rc script struc set <x y z> <childId>` - create or replace a simple
  one-child `strucGen` marker pointing at a loadable structure.
- `/rc script struc list set <x y z> <listId>` - switch the spawner marker to
  legacy structure-list mode, where runtime picks from an existing RC weighted
  structure list. This places/edits a marker reference; it does not edit list
  membership.
- `/rc script struc list clear <x y z>` - clear structure-list mode and return
  to simple child mode.
- `/rc script struc shift|front|transform ...` - edit the spawner's child/list
  shift, front direction, rotation, and mirror flag.
- `/rc script trigger spawn|redstone <x y z> <true|false>` - toggle whether
  the marker runs when placed with its parent structure or when powered.
- `/rc struclist list [filter]` - discover named weighted child pools and show
  authored versus dependency-eligible member counts and weights for this
  instance.
- `/rc struclist show <listId>` - show every child membership, weight, shift,
  front, source, dependency eligibility, and stable entry key. Reads do not
  require operator level 4.
- `/rc struclist add <listId> <childId>` - add an editable loaded child with
  default weight `1`, shift `0 0 0`, and front `north`. Adding the first member
  creates the list.
- `/rc struclist weight <listId> <childId> <weight>` - set a positive finite
  membership weight.
- `/rc struclist shift <listId> <childId> <x> <y> <z>` - set the child shift.
- `/rc struclist front <listId> <childId> <north|east|south|west>` - set the
  child front.
- `/rc struclist remove <listId> <childId>` - remove the membership; removing
  the last member makes the list disappear. Membership mutations require
  permission level `4` and are active immediately without `/rc reload`.

Structure-list membership is legacy `structureList` metadata inside each child
`.rcst`; there is no separate list file. Pair-based commands reject ambiguous
imported duplicates. Use `Auth -> Script -> Pool` to select a specific duplicate
entry. Bundled members are visible but read-only until copied/exported under an
editable id.

Named-list selection honors each child's top-level `dependencyExpression`.
Unmatched or unsupported members remain visible but are excluded before front
and weighted selection; if none remain, the marker places no child and reports
the excluded children. Use `Structs -> Gen -> Expr -> Dep`, or run
`/rc gen expr dependency <childId> <expression>`. The condition is child-wide,
not membership-specific. Direct `Set Child` and explicit/manual placement do not
use this named-list eligibility filter.
- `/rc loot list [filter]` - list bundled and user `.rcig` inventory
  generators.
- `/rc loot show <id>` - show one inventory generator's range and weighted
  entries.
- `/rc loot create <id>` - create an editable user `.rcig` generator.
- `/rc loot delete <id>` - delete an editable user `.rcig` generator.
- `/rc loot set range <id> <min> <max>` - set the generated item-count range.
- `/rc loot entry add <id> <item> <min> <max> <weight>` - add a weighted item
  entry to an editable `.rcig`.
- `/rc loot entry update <id> <index> <item> <min> <max> <weight>` - replace one
  1-based item entry.
- `/rc loot entry remove <id> <index>` - remove one 1-based item entry.
- `/rc loot container show <x y z>` - inspect RC or vanilla loot data on one
  container.
- `/rc loot container rcig <x y z> <id> [multi|single|component]` - place an RC
  inventory-generator marker into one container. The marker is captured when the
  structure is exported.
- `/rc loot container vanilla <x y z> <lootTable> [seed]` - assign a normal
  vanilla/datapack loot table to one container.
- `/rc loot container clear <x y z>` - remove RC generator markers and vanilla
  loot-table NBT from one container.
- `/rc undo` covers committed structure placement, clipboard paste, and marker
  painting. It does not cover worldgen, selection edits, export, metadata edits,
  preview creation, or cancel.
- The `/rc gui` Auth tab has `Tools`, `Select`, `Clipboard`, `Markers`, `Script`, and `Loot`
  pages for
  grabbing selector/inspector items, precise coordinate entry, here/look selection, one-face resize, copy, paste
  preview, `.rcst`/`.schem` export, common marker fills, command/simple or
  structure-list pool/marker authoring, vanilla loot-table assignment, and
  simple `.rcig` inventory-generator editing. The Script page is split into
  `Target`, `Cmd`, `Spawn`, `Pool`, and `Xform` subpages; the Loot page is split into
  `Box`, `Van`, `Gen`, and `Entry` subpages.
- `/rc export <id>` - write the current selection to
  `config/reccomplex/structures/active/<id>.rcst`. The resulting `.rcst` is
  marked for generic modern block-entity/entity NBT passthrough. Export does not
  open a generation dialog; select the new structure and configure it under `Gen`
  afterward. User `.rcst`
  files are loaded from loose files directly under `structures` and from
  `structures/active`; move files to `structures/inactive` to keep them
  installed but disabled.
- `/rc schem export <name>` - write the current selection to a Sponge `.schem`
  under `config/reccomplex/schematics`. Sponge exports remain on the legacy-safe
  schematic import path when re-imported.
- `/rc schematic ...` - long alias for `/rc schem ...`.
- `/rc gen show <id>` - show compact natural-generation metadata.
- `/rc gen validate <id>` - check whether a structure is ready for the current
  supported worldgen path.
- `/rc gen preset surface <id> [category]` - write an Overworld surface natural
  generation preset for an exported structure.
- `/rc gen preset underground <id> [category]` - write the current supported
  buried/underground-style natural preset for an exported structure.
- `/rc gen preset nether_surface <id> [category]` - write a Nether surface
  natural generation preset for an exported structure.
- `/rc gen preset sapling <id> <oak|spruce|birch|jungle|acacia|dark_oak> [single|2x2]`
  - write a sapling replacement preset for an exported structure.
- `/rc gen preset decoration <tree|big_mushroom|cactus|desert_well|fossil> <id>`
  - write a vanilla decoration replacement preset for an exported structure.
- `/rc gen preset village <id> [any|plains|desert|savanna|taiga|snowy]`
  - write a bounded village-piece preset for an exported structure.
- `/rc gen natural category <id> <category>` - update only the natural
  generation category.
- `/rc gen natural weight <id> <weight>` - update only the natural generation
  weight.
- `/rc gen natural baseline <id> <y>` - update only the natural baseline Y.
- `/rc gen natural dimension <id> <overworld|nether|end|all>` - update only
  the natural dimension preset.
- `/rc gen sapling weight <id> <weight>` - update a sapling generation entry's
  weight without replacing its sapling type, pattern, or shift.
- `/rc gen decoration weight <id> <weight>` - update a decoration generation
  entry's weight without replacing its decoration type or shift.
- `/rc gen village front <id> <north|east|south|west>` - set the village front
  direction.
- `/rc gen village shift <id> <x> <y> <z>` - set the village-piece spawn shift.
- `/rc gen village weight <id> <weight>` - set the village-piece generation weight.
- `/rc gen village cap <id> <max_per_village>` - set the per-village cap.
- `/rc gen village biome <id> <any|plains|desert|savanna|taiga|snowy>` - update
  the village biome expression preset.
- `/rc gen shift <id> <x> <y> <z>` - set the sapling/decoration trigger anchor
  relative to the exported structure's lower corner.
- `/rc gen clear <id>` - remove generation metadata from an exported structure.
- The `/rc gui` generation panel is split into `Nat`, `Trig`, `Village`,
  `Rules`, `Xform`, `Meta`, and `Fix` pages. It can edit natural fields,
  sapling/decoration weights and shifts, village-piece fields,
  simple biome/dimension allow/block rules, transformer presets, clear, validate,
  rotatable/mirrorable/blocking flags, and global missing-block replacement
  rules without typing the full commands. Its Meta page can make an editable
  config copy of a bundled/template structure, then edit authors, comment, and
  weblink on that copy. Bundled structures remain read-only for metadata edits,
  while Fix rules are global and non-destructive.
- GUI text fields are explicit-apply fields. Type the value, then press the
  matching button; invalid numbers or ids are rejected locally and server-side
  before any metadata is written.
- `/rc gen rules show <id>` - show common biome/dimension allow/block rules in
  plain language. Slash-qualified ids are supported for this read-only view.
- `/rc gen rules natural biome allow|block <id> <biome|#tag|alias>` - add
  natural biome rules without expression syntax. Examples: `plains`, `forest`,
  `minecraft:badlands`, `#minecraft:is_forest`.
- `/rc gen rules natural dimension allow|block <id> <overworld|nether|end|all|dimension_id>` -
  add natural dimension rules.
- In `/rc gui -> Gen -> Rules`, use the same simple names directly in the
  fields: `forest`, `plains`, `#minecraft:is_forest`, `overworld`, `nether`,
  `end`, or `all`. Raw `$FOREST`-style expressions still work through
  `/rc gen expr`, but are not needed for common rules.
- `/rc gen rules decoration biome|dimension allow|block|clear ...` - edit the
  same friendly rules for decoration entries.
- `/rc gen rules village biome <id> <any|plains|desert|savanna|taiga|snowy|biome|#tag|expression>` -
  set a village biome rule. `block` is the preferred user-facing word; older raw
  expression commands still use `deny`.
- `/rc gen expr show <id>` - show dependency, biome, and dimension expression
  summaries. This is the advanced/raw view.
- `/rc gen expr dependency <id> <expression>` - set the top-level dependency
  expression used by generation catalogs. Manual placement and read-only browse
  commands still work when this expression is false.
- `/rc gen expr natural biome clear|allow|deny|weight <id> ...` - edit natural
  biome rules. The `weight` form is
  `/rc gen expr natural biome weight <id> <weight> <expression>`.
- `/rc gen expr natural dimension preset|clear|add <id> ...` - edit natural
  dimension rules. The `add` form is
  `/rc gen expr natural dimension add <id> <weight> <expression>`.
- `/rc gen expr decoration biome clear|allow|deny|weight <id> ...` - edit
  decoration biome rules.
- `/rc gen expr decoration dimension preset|clear|add <id> ...` - edit
  decoration dimension rules.
- `/rc gen expr village biome <id> <expression>` - set a custom village biome
  expression when the village biome preset command is not enough.
- Dependency expressions support blank, mod ids, `mod:<id>`, `$<id>`, `!`,
  `&`, `|`, and parentheses. Unsupported old registry/structure dependency
  tokens are reported and evaluate false for generation.
- `/rc transform show <id>` - summarize supported transformer metadata.
- `/rc transform preset blend <id>` - add terrain blending for natural-floor
  markers.
- `/rc transform preset natural_air <id>` - add natural-air cleanup for
  natural-air markers.
- `/rc transform preset preserve_air <id>` - preserve ordinary exported air
  instead of clearing existing world blocks.
- `/rc transform preset ruins <id> [light|medium|heavy]` - add a conservative
  ruins decay preset.
- `/rc transform clear <id> <blend|natural_air|preserve_air|ruins|all_supported>`
  - remove supported transformer metadata while preserving unknown old
  transformer data.
- `/rc transform edit <id> <blend|natural_air|preserve_air|ruins> <field> <value>`
  - tune supported transformer details on exported config structures. Missing
  supported transformer entries are created from safe preset defaults first.
  `blend` and `natural_air` support `distance`, `randomization`, `source`, and
  `dest`; `preserve_air` supports `source` and `dest`; `ruins` supports
  `direction`, `min_decay`, `max_decay`, `chaos`, `density`, `gravity`,
  `erosion`, `vines`, and `cobwebs`.
- Examples: `/rc transform edit TestHouse blend distance 6`,
  `/rc transform edit TestHouse natural_air randomization 0.35`,
  `/rc transform edit TestHouse ruins max_decay 0.7`, and
  `/rc transform edit TestHouse ruins gravity true`.
- `/rc gui -> Gen -> Xform` exposes the same supported subset through
  `Summary`, `Blend`, `Air`, and `Ruins` pages. Full arbitrary old transformer
  tables and ordering UI remain deferred.
- `/rc metadata ...` or `/rc meta ...` - edit basic metadata and simple natural
  generation metadata for exported structures. These are lower-level aliases
  behind the friendlier `/rc gen` workflow.

## Worldgen And Diagnostics

- `/rc worldgen status`, `balance`, `catalog`, `inspect <id>`, `queue`,
  `recent`, and `stats` - inspect natural generation, frequency knobs,
  deferred placement, compatible surface-footprint mode, native locate planning,
  live diagnostic counters, planned-store cap skips/priority evictions, and
  optional dev watchdog counters.
- `/rc whatisthis` or `/rc here` - identify recorded RC structure footprints at
  your current position. `/rc whatisthis at <x y z>` and `/rc here at <x y z>`
  check an exact block position. V1 is RC-only and uses placement records, so it
  reliably covers RC structures generated or placed after this feature is
  installed; it does not backfill old worlds or identify vanilla structures.
- `/locate structure reccomplex:natural_structure` - use vanilla locate to
  predict the nearest aggregate RC natural structure candidate, including in
  unexplored terrain. The native structure shell accepts the legacy `.rcst`
  placement body through RC's compatibility/deferred worldgen pipeline, parking
  it durably if the live queue is busy. Native
  locate mode anchors RC natural placement to the located chunk center so the
  visible body should generate around the reported X/Z for surface structures.
- `/rc locate <structureId> [family] [radius <chunks>]` - start a tick-sliced
  predictive locate search for a specific RC natural structure without
  registering each `.rcst` as a vanilla structure. Exact matching is the default;
  `family` also accepts numbered variants such as `StonePlantMound1` without
  accepting unrelated prefixes like `StonePlantBoulder`. Use `/rc locate status`
  or `/rc locate cancel` while the server-thread search continues. The specific
  locate search now does cheap aggregate selection first and only runs expensive
  terrain/body planning when aggregate generation selected the requested target.
  It uses a small global per-tick budget and backs off while the server is behind,
  so rare searches may take longer but should not monopolize chunk loading. Output
  distinguishes exact/family matches, no target selections, target terrain
  rejects, and nearest aggregate candidates that selected a different RC id.
- `/rc generate <id> [at <x z>] [y <y>] [rotate <0-3>] [mirror]` - explicitly
  plan one legacy natural `.rcst`. For player-run commands this matches old
  `/#gen` operation semantics: it shows a capped client ghost first, then
  `/rc confirm` queues the frozen planned origin and transform through the
  normal deferred/ledger worldgen placement pipeline. `/rc cancel` clears the
  preview. Console and other non-player command sources queue immediately
  because they cannot receive a client preview. Without `at`, the player's
  current X/Z is used. The X/Z is an old-style centered surface anchor, not the
  lower structure corner. `y <y>` uses the supplied lower Y and bypasses surface
  selection; otherwise the structure's natural placer, build-height checks, and
  conformity checks are still used. This admin command ignores automatic
  selector gates such as global worldgen enablement, weights, biome/dimension
  eligibility, spacing, and allowed-structure lists. If the entire affected
  chunk span is already loaded when the work is queued, ledger-eligible requests
  queue directly as loaded deferred work so the active fast lane can start
  immediately. It is natural-generation only in this pass.
- `/rc generate preview <id> [at <x z>] [y <y>] [rotate <0-3>] [mirror]` -
  explicit spelling for the same player preview flow. The ghost is advisory and
  sampled for large structures, while final safe-write and ensureBlocks checks
  still happen during deferred placement.
- `/rc worldgen rates <structureId>` - show static selector inputs for one RC
  natural structure: category, generation weight, current biome/dimension
  weights, config multipliers, and estimated pre-terrain odds. These are not
  visible-placement odds because terrain checks, durable parking, queue resume,
  and final placement happen later.
- `/rc worldgen sample <structureId> [family] [detail] [radius <chunks>]` - run a
  tick-sliced diagnostic scan over the full requested radius. It counts normal
  aggregate RC selections, selections of the requested exact/family target,
  target terrain rejects, accepted target placements, and the nearest different
  aggregate candidate. `detail` prints capped surface-spread parity diagnostics
  for rejected target candidates. Use `/rc worldgen sample status` or
  `/rc worldgen sample cancel` while it runs.
- `/rc worldgen survey <structureId> [family] [detail] [radius <chunks>] [centers <count>] [spacing <chunks>]` -
  run several sample windows across separated areas of the current world seed
  and aggregate selector, biome, and terrain-reject evidence. Use
  `/rc worldgen survey status` or `/rc worldgen survey cancel` while it runs.
- `/rc sapling status` and `catalog` - inspect sapling replacement.
- `/rc decoration status`, `catalog`, and `probe [x y z]` - inspect decoration
  replacement.
- `/rc vanilla status`, `catalog`, `inspect <id>`, and `probe [x y z]` - inspect
  the bounded village-piece bridge.
- `/rc script show <x y z>` - inspect one RC script marker block.
- `/rc scriptblocks nearby [radius]` - find preserved RC script/action blocks.

The public command surface uses the clean names above. Internally, the mod still
uses the same `.rcst` compatibility pipeline for bundled and newly exported
structures.

## Changing Frequency

- `/rc worldgen balance` - show the main global controls and config file path.
- `/rc gui` -> `Balance` - edit the same global controls in game.
- `worldgen.includeBundledStructures` controls whether bundled jar/dev-resource
  `.rcst` files are included in default RC worldgen catalogs and shown in
  `/rc gui -> Structs`. Turning it off does not block explicit command/manual
  placement, copying, checks, or audits by id; explicit
  `worldgen.allowedStructureIds` entries can still opt bundled natural structures
  back in.
- Natural structures use `worldgen.rarity`; higher values are rarer, and
  `1` is the default RC rate.
- Natural multipliers use `worldgen.balance`: `naturalFrequencyMultiplier`
  controls all natural attempts, while `decoration`, `frequent`, `adventure`,
  `rare`, and `ultrarare` tune RC categories. `0` disables, `0.5` halves,
  `1` is normal, and `2` doubles.
- Saplings use `saplings.triggerChance` and `saplings.baseWeight`.
  Lower chance/weight means fewer RC sapling replacements.
- Decorations use `decorations.*BaseWeight`; higher values make RC
  decorators win more often. RC tree decoration first passes
  `decorations.treeChunkChance` (`0` disables RC decoration trees, `0.10`
  means roughly 10% of otherwise tree-eligible chunks, `1` restores the
  current capped behavior) and is then capped to one accepted replacement
  attempt per chunk to keep dense forests from flooding deferred placement.
- Villages use `villages.attemptChance` to softly gate eligible village chunks
  before selection, then `villages.baseWeight` to decide how strongly RC pieces
  compete when attempted.
- Individual exported structures keep their own `.rcst` weights. Use `/rc gen
  show <id>`, `/rc gen natural weight <id> <value>`, or `/rc gui` -> `Gen`.
- Surface placement compatibility uses `worldgen.compatSurfaceFootprint=true`
  by default. This measures surface roughness with the old RC safe-footprint
  shape around the trigger chunk before the full liquid/conformity checks run.
  `worldgen.maxSurfaceDelta` is the single roughness cap for supported RC
  surface placement styles, default `7`, matching the maintained original RC;
  lower values provide stricter rough-terrain rejection.
  Set it to `false` only when comparing against the stricter full-footprint port
  behavior.
- Optional natural spacing is edited from `/rc gui` -> `Balance` -> `Spacing`
  or `/rc worldgen balance set spacing ...`. It prevents accepted RC natural
  structures from starting too close to earlier RC natural structures. It is
  disabled by default and does not affect manual placement, saplings,
  decorations, villages, schematics, or scripted children.
- Command examples:
  - `/rc worldgen balance set bundled false`
  - `/rc worldgen balance set natural multiplier 0.5`
  - `/rc worldgen balance set natural rarity 2`
  - `/rc worldgen balance set natural category frequent 2`
  - `/rc worldgen balance set decorations treeChance 0.1`
  - `/rc worldgen balance set decorations weight tree 2`
  - `/rc worldgen balance set spacing enabled true`
  - `/rc worldgen balance set spacing distance 8`
  - `/rc worldgen balance set spacing category rare 16`

See also:

- [Main README](../../README.md)
- [Structure Authoring Guide](structure-authoring.md)
- [Configuration And Troubleshooting](configuration-and-troubleshooting.md)
