# Configuration And Troubleshooting

This guide covers normal worldgen tuning, config updates, deferred-placement
diagnostics, and the most common support problems in Recurrent Complex Volts.

For structure creation, see the [structure authoring guide](structure-authoring.md).
For exact command syntax, see the [command reference](command-reference.md).

## Config Location And Lifecycle

The common config is relative to the active Minecraft instance or dedicated
server directory:

```text
config/reccomplex-common.toml
```

Do not assume a development checkout's `run/config` or `run/client/config` is the
instance currently being launched. `/rc worldgen balance` prints the absolute
config path used by the running game.

The GUI `Balance` tab and `/rc worldgen balance set ...` apply supported balance
changes live and save the file. `/rc reload` reloads structure and generation
catalogs after `.rcst`, `.rcig`, or schematic file changes; it is not a general
TOML reload command.

When editing TOML manually, stop the game or server first. Existing generated
config files retain their stored values when a mod update changes a default. To
adopt every current default, back up or rename `reccomplex-common.toml` while the
game is stopped and let RC create a new file at next launch. Reapply intentional
custom settings afterward.

Older `legacyWorldgen`, `legacySaplings`, `legacyDecorations`, and
`legacyVanilla` public section names are migrated where possible.

## Important 0.7.0.0 Defaults

| Setting | Default | Meaning |
| --- | ---: | --- |
| `worldgen.enabled` | `true` | Enables RC natural structures. |
| `worldgen.includeBundledStructures` | `true` | Includes bundled structures in normal catalogs and the GUI. |
| `worldgen.rarity` | `1` | Extra denominator; higher is rarer. |
| `worldgen.balance.naturalFrequencyMultiplier` | `1.0` | Main overall natural-frequency control. |
| `worldgen.minSpawnDistance` | `96` | Minimum horizontal blocks from world spawn. |
| `worldgen.maxSurfaceDelta` | `7` | Maximum sampled terrain height spread for supported surface placers. |
| `worldgen.spacing.enabled` | `false` | Optional RC-to-RC natural spacing. |
| `saplings.triggerChance` | `1.0` | Chance an eligible sapling event may try RC. |
| `saplings.baseWeight` | `0.2` | RC competition weight for eligible sapling events. |
| `decorations.treeChunkChance` | `0.10` | Chunk-level gate before an RC tree replacement attempt. |
| `decorations.treeBaseWeight` | `0.03` | RC tree competition weight after that gate. |
| Other decoration base weights | `0.2` | Mushroom, cactus, desert-well, and fossil competition weights. |
| `villages.attemptChance` | `0.35` | Soft gate for eligible village chunks. |
| `villages.baseWeight` | `10.0` | RC village-piece competition weight after the gate. |
| `villages.budget` | `8.0` | Aggregate accepted RC-piece cost cap per detected village. |
| Active/complement fast lanes | enabled | Up to `1024` operations and approximately `8 ms` per eligible task/tick. |

## Tune Overall Structure Frequency

Use `/rc gui -> Balance` for ordinary changes.

Start with `Natural multiplier`:

- `0` disables natural attempts through the multiplier path.
- `0.5` is approximately half the normal attempt pressure.
- `1` is normal.
- `2` is approximately twice the normal attempt pressure.

`Rarity` is the older extra denominator. `1` is normal and larger integers are
rarer. Both settings remain for compatibility, but most users should leave
rarity at `1` and use the multiplier for straightforward overall tuning.

Category multipliers tune natural structures marked `decoration`, `frequent`,
`adventure`, `rare`, or `ultrarare`. Individual `.rcst` generation weights then
control competition among eligible structures; they are not direct percentages.

Optional spacing prevents accepted RC natural structures in the same category
from starting too close together. It is disabled by default and does not affect
manual placement, saplings, vanilla-decoration replacement, villages,
schematics, or script children. The natural `decoration` category is excluded
unless `worldgen.spacing.includeDecoration` is enabled.

## Bundled And User Structures

`worldgen.includeBundledStructures=false` removes bundled jar structures from
default worldgen catalogs and hides them from `Structs`. It does not delete them
or block direct commands, checks, audits, copying, or manual placement.

`worldgen.allowedStructureIds` is an optional allowlist. Empty means the normal
catalog is allowed. A non-empty list restricts natural generation to matching
ids and can explicitly opt bundled natural structures back in even when bundled
catalog inclusion is off.

User structures normally live under:

```text
config/reccomplex/structures/active
```

Files under `structures/inactive` are ignored. Loose files directly under
`structures` and the older `active/structures` layout remain readable for
compatibility.

## Terrain Acceptance

`worldgen.maxSurfaceDelta` controls the maximum sampled height spread accepted by
supported natural surface placers. The default is `7`. A smaller number rejects
more slopes; a larger number accepts rougher terrain.

`worldgen.compatSurfaceFootprint=true` uses the old RC-style sampled safe
footprint rather than requiring the complete rectangular footprint for the first
roughness check. Large structures therefore sample a bounded subset instead of
every X/Z point, although later conformity and liquid checks still apply.

`worldgen.minSpawnDistance` keeps natural RC structures away from the initial
world spawn. It does not affect `/rc generate` or manual placement.

## Saplings, Decorations, And Villages

These are separate bridges, not natural-category multipliers:

- Saplings first use `triggerChance`, then `baseWeight` to compete with vanilla.
- RC trees first use `treeChunkChance`, then `treeBaseWeight`, with at most one
  accepted RC tree-replacement attempt per eligible chunk.
- Other decoration types use their matching base weights.
- Villages first use `attemptChance`, then `baseWeight`. The per-village budget
  is a safety cap, not the normal density control.

Lower `villages.attemptChance` when average villages contain too many RC pieces.
Lowering the budget instead creates a harder cap on the total accepted RC-piece
cost in each detected village.

## Deferred Placement And Fast Lanes

Worldgen selection happens during chunk generation, but old multi-chunk RC
structures are placed later on the server thread when writes are safe.

- Plain eligible natural structures can use the chunk-complement ledger. Each
  loaded chunk receives only its own writable slice.
- Already-started loaded natural, village, and script-child placements can use
  the active fast lane.
- Already-started loaded ledger complement slices can use the complement fast
  lane.
- Decorations stay on conservative pacing.
- RC never force-loads missing chunks for these fast lanes. Waiting work resumes
  when required chunks become loaded and safe.

The fast-lane operation and time limits are performance safeguards. Raising them
can finish visible structures sooner but increases server-tick risk. Do not tune
queue caps or fast lanes merely to change spawn frequency.

## Diagnostic Commands

```text
/rc status
/rc worldgen status
/rc worldgen balance
/rc worldgen catalog
/rc worldgen inspect <id>
/rc worldgen rates <id>
/rc worldgen queue
/rc worldgen recent
/rc here
```

For focused bridge checks:

```text
/rc sapling status
/rc decoration status
/rc decoration catalog
/rc decoration probe
/rc vanilla status
/rc vanilla inspect <id>
/rc vanilla probe
```

## No RC Natural Structures Appear

1. Confirm that you are exploring newly generated chunks. Installing or changing
   RC does not regenerate old chunks.
2. Run `/rc worldgen status` and confirm natural generation is enabled.
3. Run `/rc worldgen catalog` and confirm the catalog is not empty.
4. Check bundled inclusion and `allowedStructureIds` with
   `/rc worldgen balance`.
5. Check whether an old config retained an unexpectedly high rarity or low
   multiplier.
6. Move beyond `worldgen.minSpawnDistance` from world spawn.
7. Use `/rc worldgen recent` to distinguish selection, terrain rejection,
   spacing, and queue behavior.

## One Particular Structure Does Not Appear

Run:

```text
/rc check <id>
/rc gen validate <id>
/rc worldgen inspect <id>
/rc worldgen rates <id>
```

`/rc generate <id>` is useful for testing its natural placer at your current X/Z
without automatic frequency, biome, dimension, spacing, allowlist, or global
worldgen gates. A successful explicit preview does not prove that automatic
selector eligibility is configured correctly.

## Trees Or Decorations Do Not Appear

Use `/rc decoration probe` at the surface location and inspect the decoration
catalog. Check `decorations.enabled`, `treeChunkChance`, the matching base weight,
the structure's biome rules, and whether its saved `ensureBlocks` ground checks
can pass.

`/rc here` does not identify most decoration trees or sapling replacements by
default because their `recordFootprints` settings default to false.

## Village Pieces Do Not Appear

Use `/rc vanilla probe` in or near a newly generating village and
`/rc vanilla inspect <id>`. Check `villages.enabled`, `attemptChance`, metadata
biome eligibility, per-piece cap, and aggregate budget.

RC uses a bounded compatibility bridge near detected modern village pieces, not
true jigsaw-pool injection. A village can validly receive no RC piece, and failed
surface, overlap, ensure-block, or budget checks can reject an attempted piece.

## A Structure Appears Slowly Or In Pieces

Run `/rc worldgen queue` while the affected chunks are loaded. The output shows
pending, planned, active, ledger, and separate fast-lane counters.

- Loaded active or complement work should show fast-lane activity.
- Work pauses when chunks unload or safe-write checks cannot pass.
- Decoration placements are intentionally conservative.
- Extremely large builds are not instant regional jobs and may complete as more
  chunks become loaded.

If ordinary chunks also stop loading, capture `logs/latest.log` before leaving
the world, along with several `/rc worldgen queue` and `recent` outputs. Reloading
the world may clear a Minecraft chunk-loading stall, but the log from the stalled
session is the useful diagnostic evidence.

## `/rc here` Finds Nothing

The command reads RC footprint records; it does not scan blocks and guess which
structure they came from.

- Worlds generated before footprint recording are not backfilled.
- You must stand inside the recorded three-dimensional bounds.
- Sapling and decoration footprints are disabled by default.
- `worldgen.maxStructureFootprintEntries=0` disables new footprint records.
- Vanilla structures are outside this RC-only index.

## Bundled Structures Disappeared From The GUI

Check `worldgen.includeBundledStructures`. Turning it off hides bundled entries
from `Structs` as well as default worldgen catalogs. User structures remain.
Bundled ids are still available to explicit commands and can be copied by id.

## Imported Blocks Become Air

Run `/rc missing one <id>` or `/rc missing dump`. Missing mod content deliberately
falls back to air instead of crashing.

Use `Gen -> Fix -> Set Global` for a reusable replacement, or make an editable
copy and use `Apply Here` to rewrite only that structure. Do not map an unknown
machine block to an unrelated block merely to remove a warning.

## Reporting A Problem

Include:

- exact Minecraft, loader, and RC Volts versions;
- `logs/latest.log` and any crash report;
- whether the config was regenerated or retained from an older version;
- world seed, dimension, and coordinates for worldgen issues;
- structure or schematic id;
- `/rc worldgen status`, `queue`, and `recent` output when relevant;
- a screenshot for visible placement or GUI problems;
- a minimal affected `.rcst` when it can be shared.

Do not repeatedly reproduce a worldgen stall before saving the first useful log.

See also:

- [Main README](../../README.md)
- [Command Reference](command-reference.md)
- [Structure Authoring Guide](structure-authoring.md)
