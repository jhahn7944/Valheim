# DogHaus Valheim — handoff from the "Best Valheim mods" session

> **Recommendations only. Verify with the user before making any change.** Nothing here is approved.
> Present the plan to the user and get an explicit yes before editing, uploading, restarting, or posting anything.

The original session ran on the Windows PC (working files in `S:\Files\Regular\Personal\Valheim`,
r2modman profile `DogHaus Valheim v2.3`, server on kineticpanel `936968c5`). It stopped at
19:03 on 2026-09-30 when it hit a usage limit, mid-way through pushing settings to the server.
This file records exactly what finished, what didn't, and what to run next.

## The request being worked (2026-09-30 18:54)

1. Terrain fixes — keep the 2.5 m per-dig cap, apply the exclusion list, dig only when the blow hits the ground.
2. Year length: an integer proportional to 365 that splits evenly into 4 seasons.
3. Fix the seasonal global keys.
4. Rebalance raids so what spawns fits the season (Deep North and Ashlands left alone).
5. Deep-dive every mod's docs with subagents; report ways to make the mods feel integrated.
6. (follow-up, 19:12) How do Cold vs Freezing work — will winter be unplayable early-game?

## Status

| # | Item | Local PC | Server | Notes |
|---|------|----------|--------|-------|
| 1 | CreatureTerrainDamage 1.7.0 | Built, deployed to local profile | **Not uploaded** | 16 creatures + `Bat_Swamp` set to 0 depth; ground-snap fix; new `Ground-Only Check Radius (m)` = 2.5 |
| 2 | 72-day year (4 × 18); **changed from 364 (4 × 91)** by the user; confirm before applying | Written to `Default settings` — **will be overwritten** | Written to `Default settings` — **already reverted once** | See "Why the Seasons values keep reverting" |
| 3 | `Enable setting seasonal Global Keys = true` | Done | Done, verified | Lives in `shudnal.Seasons.cfg`, which persists |
| 4 | Seasonal raid gating | `custom_raids.raids.cfg` regenerated: 214 raids, 46 vanilla clones, vanilla share 39–42 % every season | **Not uploaded** | 10 archetypes gated to one season (below) |
| — | Oslo night lengths (40/25/58/72 %), 24-min day | Default settings only | `Day length in seconds = 1440` persisted; night lengths **reverted** | Discord v2.6 post already tells players about Oslo nights |
| 5 | Integration deep-dive | — | — | Done in the cloud session: `INTEGRATION-REPORT.md` |
| 6 | Cold vs Freezing | — | — | Answered below |

Seasonal raid assignment (`raid_seasons.py`): Spring — Downpour · Summer — Stormcalled, StillAir, Daybreak ·
Fall — MistWalkers, MistShades, DeadAir · Winter — FirstSnow, Darklands, Nightfall. Everything else runs all year.
The old session believed `RequiredGlobalKeys` was AND-only and limited each archetype to one season. Custom Raids also has `RequireOneOfGlobalKeys` (OR), so two-season raids are possible; see `INTEGRATION-REPORT.md`. Also confirm the key case (`season_winter` vs `Season_Winter`) with `globalkeys` on the server.
Vanilla-clone batches in `raid_clones.py` were cut to 10/10/5/11/10 to hold the vanilla share near 40 %.

## Why the Seasons values keep reverting

Seasons regenerates `BepInEx/config/shudnal.Seasons/Default settings/` on **every world load**. From the README
(github.com/shudnal/Seasons):

> Do not edit files inside the `Default settings` folder directly. Any changes will be overwritten the next time you
> load a world or update the mod.
> To create custom seasonal settings: copy the file you want to edit (e.g. `Winter.json`) … paste it into the main
> `shudnal.Seasons` config folder: `BepInEx\config\shudnal.Seasons\`

The 18:39 "survived restart" check was a false positive: it read the files while the server state was still
`starting`, before Seasons had loaded. The mod then rewrote them at ~18:38–18:40, and the 19:01 read showed the
defaults (30/15/30/45 %) back.

### Fix (run from the PC session, same panel API it already used)

1. Read each `Default settings/{Spring,Summer,Fall,Winter}.json` from the server.
2. Set `daysInSeason = 18` and `nightLength` = Spring 40, Summer 25, Fall 58, Winter 72.
3. Write the result to `/BepInEx/config/shudnal.Seasons/{Season}.json` (the folder **above** `Default settings`).
4. Restart, wait for `running` **and** for a Seasons line after "Loading [Seasons 1.10.3]" in `LogOutput.log`, then
   re-read the four root-folder files and confirm 18 d / 40·25·58·72 %.
5. Do the same in the local r2modman profile so single-player/testing matches.
6. Any other override (e.g. `Custom environments.json` for the cold changes below) goes in the same root folder.

## Still to push to the server

- `BepInEx/config/custom_raids.raids.cfg` — the 214-raid seasonal build from the local profile.
- `BepInEx/plugins/DogHaus-CreatureTerrainDamage/CreatureTerrainDamage.dll` 1.7.0 (+ config if the new
  `Ground-Only Check Radius (m)` entry should be set server-side).
- The four root-folder Seasons JSONs above.
- Then rebuild the CORE/FULL packs as v2.7 (terrain mod version changed) and post them to #mod-updates,
  mentioning: 18-day seasons (72-day year), seasonal raids, creatures no longer dig when they hit a player or building.

## Cold vs Freezing — will winter be playable early?

Vanilla debuffs (Valheim wiki, weirdgloop):

| | Health regen | Stamina regen | Eitr regen | Damage |
|---|---|---|---|---|
| **Cold** | −50 % | −25 % | −25 % | none |
| **Freezing** | −100 % | −60 % | −60 % | 1 HP/s |

- **Cold never hurts you.** It only slows regen. Fire, shelter, frost resistance all clear it.
- **Freezing drains HP**, but Seasons adds a floor: *"you won't die from freezing debuff if you're not in the
  mountains or deep north. You will stay at very low hp."* So in Meadows/Black Forest/Swamp it can't kill you by
  itself, but it leaves you one hit from death in a fight.
- **Shelter downgrades Freezing to Cold; standing by fire removes both.**

What your server's winter weather actually does (from the local `Custom environments.json` winter clones):

- Cold (mostly at night): Clear, Misty, DeepForest Mist, Rain, LightRain, SwampRain, Mistlands clear/rain,
  Darklands, Heath clear.
- **Freezing**: `Snow Winter`, `SnowStorm Winter`, `ThunderStorm Winter`, `Mistlands_thunder Winter`.

So a pre-silver player in winter spends most of the time merely Cold (slower regen), and gets Freezing during
snow and storms. The catch on this server specifically: with 72 % winter nights and 18-day seasons, winter is
~7.2 real hours of play, most of it dark and Cold. A fresh character can land in a full winter with no frost gear
(Wolf armour needs silver; Lox/Feather capes are later still; Frost Resistance Mead needs Swamp bloodbags).

Early-game tools that already exist:

- **Torches** — Seasons `torchAsFiresource` makes a held torch count as fire (warm) at faster durability drain
  (0.1 vs 0.0333). Check it is `true` in `shudnal.Seasons.cfg`; this is the single biggest early-winter lifeline.
- Campfires / shelter / Rested — normal vanilla counters.
- Winter also gives faster stamina regen ("fresh winter air") and extra fire resistance.

Recommended softening, if wanted (keeps winter meaningful, removes the early-game wall):

1. In a root-folder copy of `Custom environments.json`, change `Snow Winter`, `SnowStorm Winter`, and
   `ThunderStorm Winter` from `m_isFreezing: true` to `m_isFreezing: false, m_isFreezingAtNight: true,
   m_isCold: true`. Daytime snow becomes Cold; nights in a storm still Freeze. **Check first** in
   `Custom Biome Environments.json` which biomes pull these winter clones in. If Mountain uses `Snow Winter` too,
   make Meadows/Black Forest/Swamp point at a new softened clone (e.g. `Snow Winter Lowland`) instead of editing
   the shared one, so the Mountain stays Freezing and frost gear still matters.
2. Or, milder: leave the environments alone and just make sure `torchAsFiresource = true`.

Either is a Seasons config change only — no raid or terrain work is affected.
