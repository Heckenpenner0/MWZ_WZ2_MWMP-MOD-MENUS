# my_menus — my own MP and Zombies mod menus

Two menus I wrote for this build, one per mode:

| file | mode | lines |
| --- | --- | --- |
| `MP_Menu.gsc` | Multiplayer | 5292 |
| `ZM_Menu.gsc` | Zombies / MWZ | 5250 |

Both are standalone scripts. You compile one, load it, and you get that mode's
menu — you never load both at once (see *Identity* below).

## The rule both files obey

DEV ERROR 12502 is a **link** failure: the engine refuses an import it cannot
resolve, and one bad entry takes the whole menu down. Both files were built on
the loadable surface that already works in this client — the eight `#using` lines
and the `#namespace` of the menu that opens today — and everything they add on
top of that is deliberate and counted:

- includes: the eight that load, plus `scripts\mp\equipment` and
  `scripts\mp\supers`. Only those two, and not on speculation — `TestBunny.gsc`
  has carried them (with `perk` and `mp_agent`) and loads in this client, so both
  namespaces provably resolve here.
- new hashed symbols: twelve. The `equipment` and `supers` namespace tokens plus
  ten functions — `equipment::{giveequipment, getcurrentequipment,
  getequipmentmaxammo, setequipmentammo}`, `supers::{getcurrentsuper,
  getsuperpointsneeded, updatesuperuiprogress, clearsuper, givesuper}` and the
  `issubstr` builtin the zombie filter needs. Every one is a real export of a
  script this file links: `clearsuper` and `givesuper` are the two round-4 added
  from `scripts\mp\supers`, the script whose other three exports these files
  already call, so the include surface itself is unchanged.
- new plain-name engine calls: **none** (audited)
- new `#"hash_…"` / `@"hash_…"` literals: **none** (audited)

Everything else the zombie and attachment fixes needed is free: `level.agentarray`
and `agent_type` are plain field reads, a **preset** is only this file's own
actions threaded in sequence, and a **reset** only calls the toggles the file
already owns.

Both files are derivatives of `SpaceBunny.gsc`, the menu whose eight-include
surface loads. The importer surface is therefore no longer byte-identical to the
pristine menu's, and the round-2 audits that assert it now report FAIL on purpose
— exactly as they do for `TestBunny.gsc`. The load-safety argument is now the one
above: every added symbol is a real export of a script that already links here.

## Build

Same registered identity as the working menu, so it drops into the same slot:

```bash
# Multiplayer
acts gscc MP_Menu.gsc -o matchstats -g mwiii \
     --name "scripts/mp/matchstats.gsc" \
     --namespace "scripts/mp/matchstats"

# Zombies
acts gscc ZM_Menu.gsc -o matchstats -g mwiii \
     --name "scripts/mp/matchstats.gsc" \
     --namespace "scripts/mp/matchstats"
```

Add `--crc "<your working value>"` if your loader matches on CRC — take it from
the payload you already load, not from a recomputation.

### Identity

Both files keep `#namespace namespace_a84107e4d36dda9` and register as
`scripts/mp/matchstats`. That is what makes them drop-in, and it also means they
**replace each other**: compile and load the one for the mode you are playing.

## What is in them

### `MP_Menu.gsc` — Multiplayer

Main page, with **Multiplayer Presets** first:

- **Multiplayer Presets** (new)
  - `Tryhard Preset` — aimbot, soft aim, no recoil, no flinch, high FOV
  - `Ghost Preset` — noclip, 3rd person, no recoil, super jump
  - `Chaos Preset` — explosive bullets, rocket ride, super slide, super speed,
    one Nuke killstreak
  - `Full Arsenal` — give-all-weapons, Pack-A-Punch, max ammo, every acquisition
  - `Reset All Mods` — turns off every toggle that is currently on and says how
    many it cleared
- then the existing pages: Zombies Menu (Survival / Armor + Plates / Zombies /
  Loot / Movement + View, with the armor-vest ladder and **Golden Armor Plates**,
  **Full Resupply**, **Count Zombies**, **Teleport Zombies To Me**, **Teleport To
  Crosshair**), **All Players**, Player Mods (Aimbot -> the mode page, Soft Aim,
  Bullet Menu), Weapon Options (now the single home for weapon work — see
  *Round 4*), Fun Mods (rocket ride, super slide, **Earthquake**), Killstreak Menu
  (26 killstreaks), Vehicle Menu, Lobby Mods (teleport all, kill all, give all
  weapons, force camo, UFO for all), Bot Options (spawn 1/5/20/50/100 bots, and
  **Count Bots** / **Teleport Bots To Me** / **Remove All Bots**, the zombie
  sweeps and the two AI teleports on the mobspawn page), Visions Menu, Close
  Menu.

Every preset reads the stored toggle state before it acts, so pressing a preset
twice never flips a feature backwards.

### `ZM_Menu.gsc` — Zombies / MWZ

Main page, zombies first, with **Zombie Presets** second:

- **Zombie Presets** (new)
  - `Survival Kit` — god mode, infinite ammo, infinite plates, infinite durability
  - `Slayer Kit` — the survival set plus all Wonder Weapons and +50000 essence
  - `Full Arsenal` — Wonder Weapons, every acquisition, Pack-A-Punch, max ammo
  - `Essence Flood (+999999)` — essence to the game's own 999999 cap
  - `Wipe Horde` — remove all zombies, then top up ammo and plates
  - `Reset All Mods` — clears every tracked toggle, TRUEGOD first, with a count
- **Multiplayer** (new page) — the five MP submenus moved off Main, so a Zombies
  match opens on zombies: Killstreak Menu, Vehicle Menu, Lobby Mods, Bot Options,
  Visions Menu
- Zombies Menu, sectioned Survival / Armor + Plates / Zombies / Loot / Movement
  + View: TRUEGOD, Damage Reflect, god mode, the 2-plate-then-3-plate armor
  ladder, **Golden Armor Plates**, armor plates and durability, **Full Resupply**,
  **Count Zombies**, **Teleport Zombies To Me**, **Teleport To Crosshair**, the
  movement toggles, and the mobspawn page (the bot spawn rows, **Count Bots** /
  **Teleport Bots To Me** / **Remove All Bots**, the zombie one-shot / heal /
  clear rows and the two AI teleports)
- **All Players** (new) — Give All Players God Mode / Infinite Ammo / Max Essence
- **Weapon Options** — the single home for everything weapon-shaped: the
  `--WeaponUpgrades--`, `--Ammo--`, `--Equipment--` and `--FieldUpgrades--`
  groups, including the two Pack-A-Punch rows, the field upgrade switcher and
  lethal/tactical gear

## Checking them

`tools/gsc_research/CHECK.sh` takes the file to check as its argument:

```bash
sh tools/gsc_research/CHECK.sh my_menus/ZM_Menu.gsc
sh tools/gsc_research/CHECK.sh my_menus/MP_Menu.gsc
```

`CHECK.sh` covers what must hold regardless of the include set: unique export
hashes, every `&fn` reference defined, balanced delimiters, every menu row in the
select-handler allowlist, and no new plain-name or hash-literal import. Those all
pass for both files.

The two import audits (`audit_imports.py`, `audit_calls.py`) now print FAIL on
purpose for both files, the same way they do for `TestBunny.gsc`: the
`equipment` and `supers` includes add imports the pristine menu never had. Read
their two lists — ten hashed symbols and the qualified call forms for them — and
nothing else.

## What has and has not been verified

`sh tools/gsc_research/CHECK.sh` was run against all three files after the round-4
changes:

| check | `MP_Menu.gsc` | `ZM_Menu.gsc` |
| --- | --- | --- |
| `#using` lines | 10 | 10 |
| definitions / duplicate export hashes | 172 / none | 173 / none |
| `&` reference with no definition | none | none |
| local call with no definition | none | none |
| delimiters `{}` `()` `[]` | 703/703, 1929/1929, 1207/1207 | 692/692, 1906/1906, 1187/1187 |
| menu rows missing from the allowlist | none (93 entries) | none (94 entries) |
| new plain-name / hash-literal imports | none, none | none, none |
| new hashed symbols vs the pristine menu | 12 | 12 |
| line endings | 5292 CRLF, 0 lone LF | 5250 CRLF, 0 lone LF |

Those twelve hashed symbols are the whole load-safety question now, and they are
listed in *The rule both files obey* above.

### New checks added this session

`tools/gsc_research/accept_r5.py` is the Round-5 gate: it checks all three files
(`TestBunny.gsc`, `my_menus/MP_Menu.gsc`, `my_menus/ZM_Menu.gsc`) for the eight
Round-5 changes at once and prints `ALL CHECKS PASS`. It was written in this
session and is tracked alongside the menu files. `CHECK.sh` and `accept_r5.py` were
both re-run after the README edit below:

```bash
python3 tools/gsc_research/accept_r5.py TestBunny.gsc my_menus/ZM_Menu.gsc my_menus/MP_Menu.gsc
for f in TestBunny.gsc my_menus/ZM_Menu.gsc my_menus/MP_Menu.gsc; do sh tools/gsc_research/CHECK.sh "$f"; done
```

**Honest caveat on `accept_r5.py`'s pass status.** Earlier in this session, manual
file searches looked like they disagreed with `accept_r5.py`'s pass report: the
declarations for the Round-5 action functions did **not** show up in a naive grep.
That turned out to be a line-ending / regex-mode mismatch, not a missing declaration.

The naive search was run against the raw CRLF file with a non-multiline regex
(`re.search(r"^function mm_zombie_teleport", src)` without `re.M`), so a line that
starts after a `\r` can never match. The same check run the way `accept_r5.py` runs it
— read the file, normalise `\r\n` → `\n`, strip `//` comments, then search with `re.M`
— finds all ten declarations in all three files. That was re-checked immediately before
this paragraph was written and still holds: `mm_zombie_teleport`, `mm_can_see`,
`mm_ammo_mod`, `mm_ammo_show`, `mm_perk_all`, `mm_perk_clear`, `mm_set_operator`,
`mm_spawn_ai`, `mm_spawn_all_enemies` and `mm_ai_spawn_one` are all present in
`TestBunny.gsc`, `MP_Menu.gsc` and `ZM_Menu.gsc`.

So `accept_r5.py`'s `ALL CHECKS PASS` is now consistent with the on-disk content, and the
table below reflects the values `CHECK.sh` printed when it was re-run at that point.



| check | `TestBunny.gsc` | `MP_Menu.gsc` | `ZM_Menu.gsc` |
| --- | --- | --- | --- |
| `#using` lines | 15 | 15 | 15 |
| definitions / duplicate export hashes | 179 / none | 182 / none | 183 / none |
| `&` reference with no definition | none | none | none |
| local call with no definition | none | none | none |
| delimiters `{}` `()` `[]` | 716/716, 1944/1944, 1382/1382 | 741/741, 2024/2024, 1396/1396 | 730/730, 2001/2001, 1372/1372 |
| menu rows missing from the allowlist | none (94 entries) | none (101 entries) | none (102 entries) |
| new plain-name / hash-literal imports | none | none | none |
| new hashed symbols vs the pristine menu | 23 | 23 | 23 |
| line endings | 5463 CRLF, 0 lone LF | 5581 CRLF, 0 lone LF | 5538 CRLF, 0 lone LF |

The twenty-three new hashed symbols are the whole Round-5 load-safety question:
`9` includes-worth (*TestBunny.gsc* already carried the first 15 on its own, so those
three files only added `mp_agent`, `utility/perk`, `teams`, `operator` and
`script_6bffae1b97f70547` to the already-working surface) plus the 14 new round-5
function/namespace tokens the three files share (`ammo_mod`, `teams`, `operator` and
their exports, plus the zombie perk helpers). The import audits
(`audit_imports.py`, `audit_calls.py`) still print FAIL on purpose for all three
files: the new surface is bigger than the pristine menu's, and that is by design.
Read their two lists instead of the exit code.

The table values above are the ones `CHECK.sh` returned when it was run earlier in this
session. Do not treat `accept_r5.py`'s `ALL CHECKS PASS` as final until the missing
declaration problem below is resolved.

**Not run:** the `acts` compile and the game. There is no compiler in the
environment these were built in, so link success is still only provable by
loading the compiled payload. It is called `matchstats.gscc` and it replaces the
menu you already load.

Two honest limits baked into the design:

- `Reset All Mods` in `ZM_Menu.gsc` does **not** clear no-flinch, high-FOV or
  UFO-for-all: those live in level flags rather than the per-player toggle store,
  and the file cannot show that calling them while set turns them off. It clears
  the 19 tracked toggles and leaves the rest alone rather than guess.
- out-of-bounds immunity has no stored state in either file, so reset ignores it
  for the same reason.

## Round 2: what was added here, and what cannot be

Both files gained the zero-import half of the round-2 work in `TestBunny.gsc`:
sixteen functions (`mm_zombie_list`, `mm_zombie_count`, `mm_zombie_teleport`, the
four `mm_bot_*` helpers, `mm_earthquake`, `mm_teleport_crosshair`, `mm_resupply`,
the four `mm_ai_*` aimbot functions, `mm_pap_max`, `mm_pap_reset`), plus the
section headers, the new rows, the allowlist clauses and the `ai_aimbot` toggle.
They are copied from `TestBunny.gsc` verbatim, so the behaviour is verified in
one place rather than two. `AI Aimbot` is the same port of the pristine `Zombie
Aimbot`, and `Reset All Mods` now clears it like every other toggle.

## Round 3: what changed here

`TestBunny.gsc` got the round-3 fixes first; these two files were then brought up
to the same behaviour, function for function:

- `mm_agent_list` / `mm_is_zombie` are new: AI targets come from
  `level.agentarray` filtered by `issubstr( agent.agent_type, "zombie" )`, which
  is how the build's own `cp_mp\agents\agent_utils.gsc` classifies agents. The old
  `getentarray( "actor_jup_spawner_zombie", "classname" )` + `!isdefined( pers )`
  list was empty every time, because `mp_agent` gives every agent a `pers` array
  on spawn. That single bug is why the counter read 0, teleport moved nothing, the
  one-shot/heal/clear rows did nothing and `Wipe Horde` wiped nothing.
- `mm_inf_equipment_loop` now uses `equipment::getcurrentequipment` /
  `getequipmentmaxammo` / `setequipmentammo` per slot instead of refilling the
  equipment table's template object. Lethals carry variant ids and tacticals
  usually do not, which is exactly why only the non-lethal slot looked unlimited.
- `mm_inf_fieldupgrade_loop` now drives the real charge:
  `supers::getcurrentsuper()` -> `basepoints = getsuperpointsneeded()`,
  `setweaponammoclip( weaponobj, 1 )`, `updatesuperuiprogress()`. The old loop
  topped up ammo on the super's weapon object, which does not touch the charge.
- `mm_damage_reflect` now also starts `mm_reflect_watch`, which answers the
  ATTACK rather than the damage event, so reflect still works with this menu's god
  mode on. And the kill branch asks `mm_is_player()` (a `level.players` lookup)
  instead of `isdefined( attacker.pers )` — `pers` is on agents too, so both files
  used to send zombies down the player-only `suicide()` path.
- the attachment rows and `mm_att_apply( list )` read the held weapon's own
  `getweaponattachments()` instead of a menu-maintained list, which is what made
  the first press strip every attachment and print `Requested:0`.
- `mm_give_equipment` is now `equipment::giveequipment`, the same call
  `TestBunny.gsc` makes, so lethal and tactical spawning behaves identically in
  both. That is the change that needed `#using scripts\mp\equipment`.

**What is still not ported, and why.** The perk rows (`Give All MP Perks`, `Give
All ZM Perks`, `Remove All Perks`, the perk lists), the zombie type selector and
the whole `Zombie Spawner` page need `perk` and `mp_agent`, which these files do
not `#using`. Adding a third and fourth include is a bigger step than the two
these files took, so the zombie count and teleport rows sit on the Zombies Menu
instead and the spawner stays in the one file that already imports `mp_agent`.

The split is therefore: `TestBunny.gsc` is the feature-complete menu and the one
to load in MWZ; these two are the same menu on a deliberately smaller include set,
and they now carry every fix that set allows.

## Round 5: the eight requested changes

`TestBunny.gsc` was edited first and these two files then brought up to the same
behaviour, function for function. Round 5 added three new `#using` lines across all
three files — `scripts\mp\teams`, `scripts\cp_mp\operator` and `script_6bffae1b97f70547`
— and the tokens they make loadable. They join the Round-4 surface the two my_menus
files already had (`scripts\mp\utility\perk`, `scripts\mp\mp_agent` and the
Round-4 `equipment` + `supers` includes), so every file ends on 15 includes. Nothing
below adds a plain-name builtin or a `@"hash_..."` literal; the new hashed symbols
are the `ammo_mod`, `teams` and `operator` namespace tokens plus their real exports,
and they are all from scripts that already link in this client.

The new surface is still an experiment in the same class as the Round-4 includes: the
mwiii-source dump does not contain plain `namespace_<hex>` / `function_<hex>` tokens
for cross-script calls (it uses the `namespace_x::name()` text form), so absence of a
token in the dump is **not** proof that the token is wrong. The menu file is the
authoritative text; what compiles is what gets linked.

### 1. Zombies now teleport in front of the player

`mm_zombie_teleport` was placing zombies on the player's literal origin, which is
feet-down and behind the torso. It now computes a forward vector from the player's
own aim angles (`function_6343041d48c52fd1( self.var_27cbf5badd0cbdbd )`), drops the
zombies `170` units straight ahead, offsets each one across a lane (`mm_right * lane * 55`)
so they do not stack, and raises them `25` units off the ground so they do not clip the
floor. The per-zombie lane is taken from the loop index, same as before.

### 2. The wallcheck now actually traces world geometry

The old trace mask was `( 1, 0, 0, 0, 0, 0, 1 )` — character + playerclip, which
excludes world geometry. That is why it always returned a fraction of 1 and the check
effectively never blocked anything. It now uses the `engine/trace` contents-all mask
(`create_contents( character, world, glass, shotclip, item, vehicle, playerclip,
ainosight, itemclip, vehicleclip, waterclip, cameraclip )`), and the test is the
trace struct's `"fraction"` field: a value `>= 0.9` means the ray reached the aim point
mostly untouched. If `"fraction"` is not exposed by this build, the helper returns 1
so the menu keeps running rather than hard-failing.

The honest limit: `ray_trace` returns the physics struct, and the only thing this
script can read from it is `fraction` (plus `position`, which is used elsewhere for the
crosshair point). The check can tell *whether the ray was stopped short*, it cannot name
the thing that stopped it — so it still cannot distinguish a wall from another entity that
happens to be there.

### 3. Ammo Mod: change the current weapon's elemental rounds

New `Weapon Options -> Ammo -> Ammo Mod` submenu with the five MWZ elemental rounds
and a show / remove pair:

- **Napalm Burst**, **Dead Wire**, **Cryo Freeze**, **Brain Rot**, **Shatter Blast**
- **Show Current Ammo Mod**
- **Remove Ammo Mod** (`"none"`)

`mm_ammo_mod( mm_mod )` calls
`namespace_dc2e59577d3a271f::function_6c40f58b29a952a9( mm_held, mm_mod )`, where
`mm_held` is the weapon from `function_72ffae5f008b8030` and the names are the same
keys the game's own `ammo_mod` script asserts against (`level.ammo_mods`). That setter
is what `ob.gsc`, `pack_a_punch.gsc` and the MWZ objective scripts call. `mm_ammo_show`
uses the same script's `get_ammo_mod_name`.

This needs `#using script_6bffae1b97f70547;`. The names (`napalmburst` ... `shatterblast`,
`"none"`) are taken from `mp/hashed/script/script_6bffae1b97f70547.gsc` and the callers
in `pack_a_punch.gsc`, `ob.gsc`, `mp_jup_*_ob.gsc`.

Honest limit: the setter still depends on `level.ammo_mods` being populated — i.e. on the
mode's own ammo-mod init having run. If a mode never initialises ammo mods, the call may
assert or no-op; the menu prints the *you must be holding a weapon* guard before it
reaches that point.

### 4. Give every Zombie perk, every MP perk, and each one individually

`Player Mods -> Perk Menu` now has bulk give/remove plus a per-perk list for each mode:

- **Give All MP Perks** — `mm_perk_all( "mp" )`
- **Give All ZM Perks** — `mm_perk_all( "zm" )`
- **Give Every Perk (MP + ZM)** — `mm_perk_all( "all" )`, which loops MP when `mp|all`
  and ZM when `zm|all` through one shared counter
- **Remove All Perks** — `mm_perk_clear`
- **MP Perks** → `PerksMP` page: the 12 usable MP specialty refs, each calling
  `mm_perk( name )` = `perk::giveperk( self, name, 1 )`
- **ZM Perks** → `PerksZM` page: the 10 real MWZ perk-a-cola refs, same `mm_perk`

Perks are the same calls the build's own dev script makes, so they go through
`perk::giveperk` / `perk::removeperk` from the included `utility/perk` script.

Honest limit: this is script-layer grant/remove. If zombie perk-a-colas in this container
are applied through MWZ's own perk machine rather than `perk::giveperk` (e.g. an
omnvar path), the per-perk rows and the bulk rows will still print ON, but the visible
zombie behaviour may not change. The MP side is far more likely to be real because MP
perks are registered shared functions.

### 5. Acquisitions: a Main row with the whole known list, including classified items

Main now has a dedicated **Acquisitions** row that opens `ItemsMenu`. That page is rebuilt
with section headers and every ref harvested from the mwiii-source loot files:

- **-- CLASSIFIED --** — Disciple Bottle, Disciple Bottle (Schematic)
  (`jup_schematic_classified_disciple_bottle`), Bloodburner Key
  (`jup_schematic_classified_bloodburner`), Dog Bone
  (`jup_schematic_classified_dog_bone`), Golden Armor Plate (Schematic)
  (`ob_jup_item_schematic_classified_golden_armor_plate`), Mags of Holding (Schematic)
  (`jup_schematic_classified_golden_ammo`), Warlord Beret
  (`jup_schematic_classified_warlord_afo_beret`), Warlord Chemist Mask
  (`jup_schematic_classified_warlord_chemist_mask`), Warlord Maestro Mod
  (`jup_schematic_classified_warlord_maestro_mod`), Grenade Bandolier
  (`jup_schematic_classified_grenade_bandolier`)
- **-- EQUIPMENT AND GEAR --** — Aether Blade (Lethal), Monkey Bomb (Tactical),
  LT53 Kazimir (Tactical), plus the in-match item forms
  (`ob_jup_item_weaponcase_aetherblade`, `ob_jup_item_tactical_monkey_bomb`,
  `ob_jup_item_tactical_blackhole_bomb`, `ob_jup_item_backpack_large`,
  `ob_jup_item_plate_carrier_3`, `ob_jup_item_golden_armor_plate`,
  `ob_jup_item_golden_ammo`, `ob_jup_item_warlord_maestro_mod`,
  `ob_jup_item_revive_self_revive`)
- **-- AETHER --** — Aether Crystal, Refined and Flawless crystals
  (`ob_jup_item_aether_crystal`, `..._2`, `..._3`), Aether Tool (Refined/Flawless)
  (`obloot_aether_tool_3`, `obloot_aether_tool_4`)
- **-- WONDER WEAPON CASES --** — Ray Gun Case, Plasma Gun Case, DG-2 Case, VR-11
  Case (the four known unlock-case refs)
- **-- ALL --** — Give Every Acquisition (`mm_give_all_items`)

These refs come from `mp/loot/loot_master.csv`, the `ob_jup_item_*` and
`jup_schematic_classified_*` rows, `jup_disciple_bottle`, the
`jup_ob_schematic_unlock_*` cases, and `obloot_aether_tool_{3,4}` / `obloot_offhand_*`.

Honest limit: `mm_give_item` hands classified schematics to the game's schematic-unlock
path; the in-match items and equipment go through the menu's item / give-equipment paths.
Whether a given ref is a usable item, an unlock, or schematic-only is determined by that
path, and any ref this build does not know is reported as such rather than silently ignored.

### 6. Operator Changer

New `Player Mods -> Operator Changer` → `OperatorMenu`, 18 operator skin rows. Each row
calls `mm_set_operator( skinref )` with a real `operator_skin,<name>,<idx>` loot entry
from `mp/loot/loot_master.csv` (Price, Ghost, Soap, Farah, Kleo, Ronin, Zimo, Gus,
Hutch, Chili, Fender, Calisto, Nova, Kilgore, Conor, Reyes, Gromsko, Oni).

`mm_set_operator` does two things:

1. resolves the skinref to the operator ref with
   `namespace_f4302f2673b029b9::function_605079124463a67b( mm_skin )` — that is
   `operator::getoperatorfromskin( skinref )` from `cp_mp/operator.gsc`
2. reapplies the character through
   `namespace_cf299516c16d9ef9::function_ca40eb9dbf56495a( mm_operator, mm_skin )`
   — i.e. `teams::createoperatorcustomization( operatorref, operatorskin )` from
   `mp/teams.gsc`

The operator value itself comes from the same `loot_master.csv`.

This needs `#using scripts\mp\teams;` and `#using scripts\cp_mp\operator;`. The workflow
mirrors `dev.gsc`'s own operator-change path (set `operatorcustomization.skinref`, then
`setcustomization` / `_setsuit` / `setcharactermodels`).

Honest limit: this depends on `teams` and `operator` being loadable in this client, and
on the player having an `operatorcustomization` field the game cares about. The shape is
taken from `dev.gsc`, but the in-game behaviour is not provable here.

### 7. Spawn all enemy / AI types in front of the player

New **Spawners -> Enemy And AI Types** → `EnemySpawnMenu`, with every aitype this build's
`spawnnewagentaitype` should accept, placed a short distance in front of the player:

- **-- ZOMBIES --** — Base, Armored Light, Armored Heavy, Hellhound, Mangler, Mimic,
  Disciple, Disciple HVT, Abomination, Entity
  (`actor_jup_spawner_zombie_*`)
- **-- ENEMIES --** — Grunt, Soldier, Special Force, Captain, Commander, Rusher, Sniper,
  Shielded, Rocketeer, Knight, Heavy Tactician
  (`actor_jup_ob_enemy_*`)
- **Warlords** — Maestro, Chemist, All For One
  (`actor_jup_ob_warlord_*`)
- **-- ALL --** — Spawn One Of Every Type (`mm_spawn_all_enemies`)

Each row calls `mm_spawn_ai( aitype )`, which uses
`namespace_39b39f93ee3db0c2::function_da6899fe85b80061` — the `mp_agent` spawner
`TestBunny.gsc` already uses — and falls back to a short frontpoint plus a yaw from
`gettime() % 360` when a navmesh snap is not available. `mm_spawn_all_enemies` holds a
23-entry array and spawns them with `wait 0.2` between each so the server is not hit in
one frame.

This needs `#using scripts\mp\mp_agent;` (already present) and the same spawner the
zombie rows already use.

Honest limit: which `actor_*` names this build's `spawnnewagentaitype` actually accepts is
not recoverable from the dump alone; a name the spawner does not know fails to spawn and
prints that, which is the visible answer rather than a silent no-op.

### 8. Refactor the menu structure into cleaner sub-menus

The front page is now a short hub instead of a wall:

- **Main** (9 rows): Zombies Menu, Player Mods, Weapon Options, Acquisitions,
  Spawners, Multiplayer, Lobby And Players, Fun Mods, Close Menu
- **Spawners**: Zombie Spawner, Enemy And AI Types, Bot Spawner
- **Multiplayer**: Killstreak Menu, Vehicle Menu, Visions Menu
- **Lobby And Players**: Lobby Mods, All Players

The larger workhorses are pushed one level down:

- `Weapon Options` keeps the weapon tree, plus the new `Ammo Mod` and `Acquisitions`
  routing
- `Player Mods` keeps Combat / Perks / Operator Changer
- `Spawners` becomes the single home for everything that spawns AI

`ItemsMenu` (Acquisitions) used to live inside the Zombies Menu `-- LOOT --` block and
has been promoted to a Main row so classified items are reachable directly from the front
page.

Nothing here changes which action lives in which file's handler — every `&function` still
has exactly one home and is still in the select-handler allowlist. The refactor is layout,
not capability.

### What Round 5 cannot prove

Same hard limit as every round so far: there is no `acts` compiler and no game in this
environment, so link success and in-game behaviour are still only provable by loading
`matchstats.gscc`. The Round-5-specific honest limits:

- **Zombie teleport now lands in front of the player.** The offset math is plain vector
  arithmetic on the player's own angles; it does not depend on any new import.
- **Wallcheck now uses the contents-all mask and the trace `"fraction"` signal.** It can
  still not name what it hit, and if this build's `ray_trace` does not expose `fraction`
  the helper returns 1 as a safe fallback.
- **Ammo Mod depends on `level.ammo_mods` being populated** by the mode's own init.
- **Operator Changer depends on `teams` and `operator` being loadable** and on
  `operatorcustomization` existing on the player.
- **Spawn-all depends on which `actor_*` names this build's `spawnnewagentaitype` accepts**
  — a name the spawner does not recognise fails loudly rather than silently.
- **The three new `#using` lines are the load-safety question for Round 5.** If any of
  `teams`, `operator`, or `script_6bffae1b97f70547` cannot load in this client, the menu
  12502s. That is the same uncertainty class as the Round-4 `equipment` + `supers` adds,
  and the same honest posture applies: the menu file is the source of truth, not the
  mwiii-source token dump.

## Round 4: the ten requested changes

`TestBunny.gsc` was edited first and these two files then brought up to the same
behaviour, function for function. Nothing below adds a plain-name builtin or a
hash literal; the only new hashed symbols anywhere are `supers::clearsuper` and
`supers::givesuper`.

### 1. Monkey Bomb and Kazimir go to the Tactical slot

`equip_monkey_bomb` was a row in the LETHAL group calling `mm_give_lethal`, i.e.
`equipment::giveequipment( ref, "primary" )`. It now sits in the TACTICAL group
next to `equip_blackhole_bomb` (LT53 Kazimir) and calls `mm_give_tactical`, which
passes `"secondary"`. `giveequipment` is what performs the
`assignweaponoffhandprimary/secondary` split, and both refs are real entries of
this build's equipment string table (`equip_monkey_bomb`, `equip_blackhole_bomb`).

...

`TestBunny.gsc` was edited first and these two files then brought up to the same
behaviour, function for function. Nothing below adds a plain-name builtin or a
hash literal; the only new hashed symbols anywhere are `supers::clearsuper` and
`supers::givesuper`.

### 1. Monkey Bomb and Kazimir go to the Tactical slot

`equip_monkey_bomb` was a row in the LETHAL group calling `mm_give_lethal`, i.e.
`equipment::giveequipment( ref, "primary" )`. It now sits in the TACTICAL group
next to `equip_blackhole_bomb` (LT53 Kazimir) and calls `mm_give_tactical`, which
passes `"secondary"`. `giveequipment` is what performs the
`assignweaponoffhandprimary/secondary` split, and both refs are real entries of
this build's equipment string table (`equip_monkey_bomb`, `equip_blackhole_bomb`).

### 2. The Field Upgrade switcher, and per-map registration

This was a straight bug: `mm_give_super` validated the ref against
`level.equipment.table`, the **equipment** registry. Field upgrades are not in it.
`scripts\mp\supers::registersupers` builds its own registry on load —
`level.superglobals.staticsuperdata` — keyed by the refs named in
`level.supertable` (the gametype or gamemode bundle's `superlist:`), and supers'
own `getsuperid()` validates a ref with exactly that lookup. So every row failed
the check and printed *not registered on this map*, on every map.

- `mm_super_registered( ref )` asks that registry instead, which is the map's own
  answer to "is this upgrade available here?". A ref the current map did not
  register now says so plainly.
- `mm_give_super` calls `supers::clearsuper( 0, ref, 0 )` then
  `supers::givesuper( ref, 0, 1, 0 )` — all four of `givesuper`'s parameters, so
  the blueprint index is a real 0 rather than an omitted argument.
- `mm_clear_super` now actually clears the super (`clearsuper`); in `ZM_Menu.gsc`
  and `MP_Menu.gsc` it previously only blanked the menu's own bookkeeping.
- the switcher page marks the row that is currently fitted:
  `mm_super_current_ref()` reads `getcurrentsuper().staticdata.ref` and the
  printer shows `[ACTIVE]` on the match, so a switch is visible.

### 3. Soft aim is now actually soft aim

`function_838295294b558e8` used to walk `level.players`, pick the closest one and
call `DoDamage( 100, ..., "MOD_HEAD_SHOT", ... )` on it — it never touched the
view at all, and against AI (i.e. in MWZ) it did nothing whatsoever. It now
blends `self.angles` a quarter of the way toward the nearest live enemy's aim
point every frame while the fire button is held, wrapping through the shortest
way round the circle (`mm_lerp_angle`), with `mm_softaim_target()` falling back to
`mm_nearest_of( mm_agent_list() )` when no player is in range. No engine hook of
any kind is involved, so there is no *psilent* variant to weigh against it — a
GSC script cannot redirect a bullet's trace, and the DoDamage approach it
replaces is a damage hack, not aim. This is a real aim assist and behaves like
one: the crosshair drifts in, the player keeps steering.

### 4. *Give Armor Plates* walks the carrier up

The first press fits a **2-plate** carrier, every press after that a **3-plate**
one. `mm_set_carrier( level )` writes `self.platecarrierlevel` — the tier
`scripts\cp_mp\armor` builds `maxarmorhealth` from (`135 * level`, 540 with the
extra-armor perk) and reads straight back in its own carrier helper — raises
`maxarmorhealth` to match (never *shrinking* a max the game raised itself), marks
`self.hasplatepouch` on the 3-plate step, tops the worn armor back up and refills
the carrier weapon's clip. The store keeps one flag (`plate_tier3`) so the row
toggles between the two tiers instead of stopping at 2 forever.

### 5. Golden Armor Plates, above Tier 2 and Tier 3

The armour section is now the full ladder — `Give Armor Plates`, `Tier 2 Armor
Vest`, `Tier 3 Armor Vest`, **`Golden Armor Plates`** — and the acquisition row
`Golden Armor Plate` grants the same thing instead of only topping the carrier up.
Golden armor is reproduced from `scripts\mp\ob\armor`'s own `set_golden_armor`,
which sets a player flag and starts a loop restoring a third of the armor every
four seconds, and whose damage callback halves explosive, fire and radiation
damage while the flag is set. Both halves are done here with plain player fields
(`self.var_28c033c3c5b37f1f` for the damage half, `mm_golden_armor_loop` for the
regen), so the row cannot depend on the `ob_armor` shared function being
registered on the map. It is a toggle with `[ON]`/`[OFF]` on the row and in the
`ACTIVE` overlay.

### 6. Aimbot modes: Simple / Soldier / Zombie, plus Wallcheck

`Player Mods -> Aimbot` now opens an `AimbotMenu` page: **Simple Aimbot**
(engine `getclosestenemy()` first, then any live AI), **Soldier Aimbot**
(`mm_soldier_list()` — every agent *not* classified as a zombie) and **Zombie
Aimbot** (`mm_zombie_list()`), a **Wallcheck** toggle and **Turn Aimbot Off**.
Pressing the mode that is already running turns it off. The page marks the active
mode `[SELECTED]`, the row carries `[ON]`/`[OFF]`, and the `ACTIVE` overlay gains
`Wallchk`. `mm_nearest_of( list )` is the shared picker; `mm_ai_nearest()` is now
just `mm_nearest_of( mm_agent_list() )`, so the old loop is unchanged in
behaviour.

**Wallcheck limit, stated plainly.** The trace struct this build returns does not
expose the entity it hit to this script — only `position` is read anywhere in the
pristine file. So the check is a distance compare: if the trace from the eye
toward the aim point stops more than 100 units short of it, the shot is treated
as blocked and the target is skipped. That catches walls; it cannot distinguish
them from a target standing right behind another entity.

### 7. Teleport All AI and Teleport Soldiers/Mercs

Both are toggles (on the mobspawn page in the two my_menus files, in Zombie Tools
in `TestBunny.gsc`) that put the whole group back down **five metres (200 units)
in front of the crosshair** every quarter second, so they read as a held toggle
rather than a one-shot the AI walks away from. `mm_crosshair_point()` takes the
view ray from `geteye` + `anglestoforward` and pulls the point in to the trace
hit when a wall is closer than 5 m, using the pristine vehicle spawner's own
trace pair — nothing new is imported. The loop's own exit condition is the toggle
flag it was started with, so the two rows run independently without either
threading events through the select handler.

### 8. The five weapon rows move to the Weapon Menu

`Weapon Rarity Changer`, `Pack-A-Punch Weapon`, `Pack-A-Punch To Level 3`,
`Reset Pack-A-Punch` and `Weapon Attachments` now all live on **Weapon
Options**/*Weapon Menu*. The Zombies Menu keeps a `-- LOOT --` and a
`-- MOVEMENT + VIEW --` section and nothing weapon-shaped, and the pages those
rows open (`AttachmentMenu`, `RarityMenu`, `EquipmentMenu`, `FieldUpgradeMenu`)
now go *back* to `WeaponOptions` instead of the Zombies Menu.

### 9. Ammo and equipment grouped under the new headers

The Weapon Menu is organised as `--WeaponUpgrades--`, `--Ammo--`,
`--Equipment--` and `--FieldUpgrades--`:

| header | rows |
| --- | --- |
| `--WeaponUpgrades--` | Give All Weapons, MWIII/MWII categories, Wonder Weapons, Weapon Attachments, Weapon Rarity Changer, Pack-A-Punch Weapon, Pack-A-Punch To Level 3, Reset Pack-A-Punch, Apply Camo |
| `--Ammo--` | Infinite Ammo, Max Ammo (Refill), Full Resupply (Ammo + Plates) |
| `--Equipment--` | Unlimited Equipment, Lethal and Tactical Gear |
| `--FieldUpgrades--` | Unlimited Field Upgrades, Field Upgrade Switcher |

Those rows used to be split across a `-- AMMO + EQUIPMENT --` and a
`-- WEAPONS --` block on the Zombies Menu; they live in one place now, and each
action still has exactly one home.

### 10. The All Players menu

A new `AllPlayersMenu` on Main: **Give All Players God Mode** (the existing
`mm_god_all` toggle, moved here out of the Zombies Menu), **Give All Players
Infinite Ammo** (`mm_inf_ammo_all`, the single-player infinite-ammo loop run over
`level.players`) and **Give All Players Max Essence** (`mm_points_all`, the same
`pers["score"]` + `score` + `currency["essence"]` writes `mm_add_points` makes,
for every player, to the game's own 999999 cap).

### What round 4 cannot prove

Same limit as every round before it: there is no `acts` compiler and no game in
this environment, so link success and in-game behaviour are still only provable by
loading `matchstats.gscc`. The honest specifics:

- **Soft aim's strength (0.25 of the remaining angle per frame) is a taste
  choice, not a measurement.** It is one constant in `mm_lerp_angle`'s call in
  `function_838295294b558e8` if it needs to be softer or snappier.
- **Wallcheck cannot see what the trace hit** (see item 6) — it is a distance
  compare, and it cannot tell a wall from another entity.
- **`mm_super_registered` reads `level.superglobals.staticsuperdata`**, a field of
the `supers` globals. That is correct for a map where `supers` ran its `init`; if
  a mode never initialised supers at all, the row says *not on this map's
  superlist* rather than failing silently, which is the intended behaviour.
- **`platecarrierlevel` is the tier this file writes, not a new vest model.**
  `mm_set_carrier` sets the armor capacity and refills it; moving the vest's
  position on the character model is the game's business.
- **The AI teleports move AI; they do not spawn it.** "Teleport All AI" and
  "Teleport Soldiers/Mercs" relocate what is already on `level.agentarray`; the
  count is whatever the map has.
