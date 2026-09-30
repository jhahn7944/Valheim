# Brief for the "Best Valheim mods" chat — resume here

Written 2026-09-30 by a cloud session that reviewed your transcript while you were rate-limited.
You stopped at 19:03 mid-way through pushing Seasons settings. Your context is intact; this brief adds what
was learned since and gives the remaining work in order. Details: `HANDOFF.md` and `INTEGRATION-REPORT.md` in
github.com/jhahn7944/Valheim, branch `claude/practical-darwin-dzuevo`.

## The request you're finishing (user, 18:54)

> the 2.5 cap per dig is okay, I agree with the excluded list, make those terrain fixes, if the enemy hits a
> object or person, then they should not dig only if they hit the ground, update the year length a full integer
> that will to be proportional to 365, then make sure that divides evenly into 4 seasons, fix the global seasons
> key, and adjust my raid balance so the raids that appear make sence for that season, deep north and ashland are
> fine uneffected, we can adjust them later if we want, make these fixes, then do a deep dive of all mod
> websites, use sub agents, and report back possible ways to make all these mods feel more integrated

Follow-up (19:12): *"How do the cold vs freezing effects work in this mod, since early game I dont have access to
cold resistance gear, I dont want the mod to make winter unplayable"* — answered at the end of this brief.
Give the user that answer too.

## Already done (don't redo)

- CreatureTerrainDamage 1.7.0 compiled and deployed to the **local** profile. It excludes 16 creatures plus
  `Bat_Swamp`, snaps the crater to the ground, and adds a `Ground-Only Check Radius (m)` setting (default 2.5).
- `Enable setting seasonal Global Keys = true` on the server. It's in `shudnal.Seasons.cfg` and persists.
- `Day length in seconds = 1440` on the server. It persists.
- Raid rebuild **locally**: `custom_raids.raids.cfg` has 214 raids and 46 vanilla clones, with a vanilla share of
  39–42 % in every season. `raid_seasons.py` and `raid_clones.py` are updated.
- The integration research is finished; you don't need to wait for your background agents. Results are in
  `INTEGRATION-REPORT.md`.

## Broken: why the year length and night lengths keep reverting

The user wants the year length confirmed. It is **364 days = 4 seasons × 91 days**, because 365 = 5 × 73 can't
split into four equal seasons and 364 is the nearest integer that can. Each season is then 91 in-game days,
36.4 h of play at a 24-minute day, and the full year is 145.6 h.

You wrote `daysInSeason: 91` and the Oslo `nightLength` values into
`BepInEx/config/shudnal.Seasons/Default settings/*.json`. **Seasons regenerates that folder on every world
load.** From the README:

> Do not edit files inside the `Default settings` folder directly. Any changes will be overwritten the next time
> you load a world or update the mod. To create custom seasonal settings: copy the file … paste it into the main
> `shudnal.Seasons` config folder: `BepInEx\config\shudnal.Seasons\`

Your 18:39 "survived restart" check was a false positive: it read while the state was `starting`, before Seasons
had loaded. The 19:01 read showed the defaults back. The 19:02 write to `Default settings` followed by a
restart will revert again.

## Do this, in order

1. **Seasons overrides (server).** For each of Spring, Summer, Fall and Winter:
   - read `/BepInEx/config/shudnal.Seasons/Default settings/<S>.json`;
   - set `daysInSeason` = 91 and `nightLength` = Spring 40, Summer 25, Fall 58, Winter 72;
   - write it to **`/BepInEx/config/shudnal.Seasons/<S>.json`**, the parent folder.
2. **Seasons overrides (local).** Do the same in the r2modman profile `DogHaus Valheim v2.3`.
3. **Custom Raids safety.** In the Custom Raids general config, set `StopTouchingMyConfigs = true` explicitly;
   docs disagree on its default. Do this on the server and locally.
4. **Upload to the server:**
   - `BepInEx/config/custom_raids.raids.cfg` (the 214-raid build);
   - `BepInEx/plugins/DogHaus-CreatureTerrainDamage/CreatureTerrainDamage.dll` 1.7.0.
5. **Restart and verify properly.**
   - Wait until the state is `running` **and** `LogOutput.log` shows Seasons initialised, not only "Loading
     [Seasons 1.10.3]".
   - Re-read the four parent-folder JSONs and confirm 91 d and 40/25/58/72 %.
   - Confirm the terrain mod logs `1.7.0`.
   - The browser tab froze at 19:03; reload the kineticpanel tab first and check it isn't on Discord.
6. **Check the season key string.** Run `globalkeys` in the server console (Server_devcommands is installed).
   The Seasons README gives lowercase `season_winter`, while `raid_seasons.py` writes `Season_Winter`.
   Valheim probably lowercases keys, but that isn't documented. If the printed case differs, change `KEY` in
   `raid_seasons.py` to match, regenerate, and re-upload. Also note whether any `SeasonDay_*` key exists; no doc
   mentions it.
7. **Repack and announce.** Build the v2.7 CORE/FULL packs (the terrain mod version changed) and post them in
   #mod-updates. Mention:
   - 364-day year, with 91-day seasons;
   - seasonal raids;
   - creatures no longer dig when their blow hits a player or building;
   - flyers, ghosts, slimes and small creatures never dig.
8. **Optional improvement (ask the user first).** Custom Raids also has **`RequireOneOfGlobalKeys`** (OR logic).
   The "AND-only, one season per archetype" constraint in `raid_seasons.py` is therefore not real. Two-season
   raids are possible as single raids, e.g. MistWalkers in Fall+Winter, or Stormcalled in Spring+Summer. If
   used, re-measure the vanilla share per season afterwards, as before.
9. **Report back to the user.** Give the Cold/Freezing answer below, then the top items of
   `INTEGRATION-REPORT.md`.

## Decisions to put to the user

- **Winter softening.** Options are listed in the Cold/Freezing answer below.
- **Two-season raids** (step 8).
- **Trim candidates:**
  - OdinsKingdom, which overlaps OdinArchitect and Balrond Constructions;
  - one of the two fish traps (OdinArchitect vs Balrond Shipyard);
  - AAABuildMenu, which is mostly redundant with the vanilla 1.0 build menu;
  - WhichModAddedThis, from player packs.
- **Items from client-only mods.** The ATM Shovel and the BetterArchery Leather Quiver are items that CORE-pack
  players don't have. They are probably lost if stored in shared chests. Options: move those mods to CORE, set ATM
  `Shovel=false`, or tell FULL players not to store those items in shared chests.
- **AzuCraftyBoxes.** Exclude every ship in `Azumatt.AzuCraftyBoxes.yml`; there is a documented item-duplication
  bug with ship storage. This is recommended regardless.

## Answer for the user: Cold vs Freezing in winter

| | Health regen | Stamina regen | Damage |
|---|---|---|---|
| **Cold** | −50 % | −25 % | none |
| **Freezing** | −100 % | −60 % | 1 HP/s |

- Cold never hurts you. It only slows regen.
- Freezing drains HP. Seasons adds a floor: *"you won't die from freezing debuff if you're not in the mountains or
  deep north. You will stay at very low hp."*
- Shelter turns Freezing into Cold. Standing near a fire removes both.
- On this server's winter weather:
  - most weather gives only Cold, mostly at night: Clear, Misty, Rain, SwampRain, Mistlands clear and rain,
    Heath, and Darklands;
  - Freezing comes from `Snow Winter`, `SnowStorm Winter`, `ThunderStorm Winter` and `Mistlands_thunder Winter`.
- **Real risk:** a winter is about 36 h of play with 72 % nights. A fresh character can face a whole winter before
  silver, so before wolf armour, and before Frost Resistance Mead, which needs Swamp bloodbags.
- Winter also has benefits: faster stamina regen and extra fire resistance.

Options to keep winter playable early:

1. **Minimum.** Make sure `torchAsFiresource = true` in `shudnal.Seasons.cfg`. A held torch then counts as a
   fire, at a faster durability drain of 0.1 vs 0.0333.
2. **Softer lowland snow.** Copy `Custom environments.json` to the parent `shudnal.Seasons` folder. For the
   lowland winter snow and storm clones, set:
   - `m_isFreezing: false`
   - `m_isFreezingAtNight: true`
   - `m_isCold: true`

   Check `Custom Biome Environments.json` first. If the Mountain uses the same `Snow Winter` clone, create a
   separate lowland clone for Meadows, Black Forest and Swamp instead, so the Mountain stays Freezing and frost
   gear still matters.
