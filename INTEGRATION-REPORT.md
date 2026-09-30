# DogHaus Valheim — making the mods feel like one game

> **Recommendations only. Verify with the user before making any change.** Nothing here is approved.
> Present the plan to the user and get an explicit yes before editing, uploading, restarting, or posting anything.

Research pass requested 2026-09-30 ("deep dive of all mod websites, use sub agents, and report back possible ways
to make all these mods feel more integrated"). Three research agents covered (A) Seasons + raids,
(B) building/content mods, (C) client-side/QoL mods. Their full reports are appendices A–C below.

**Sourcing caveat:** Thunderstore, Nexus and the Fandom wiki were blocked from the research environment, so the
agents read GitHub READMEs/wikis and search snippets instead. Anything marked *inferred* needs an in-game check.

---

## Corrections to work already done

1. **`RequireOneOfGlobalKeys` exists in Custom Raids.** It means "at least one of these keys", so the earlier
   "AND-only, one season per archetype" constraint in `raid_seasons.py` was wrong. Two-season raids (e.g.
   MistWalkers in Fall *and* Winter) can be written as a single raid, with no duplication and no skew to the
   vanilla share.
2. **The season key's case.** The Seasons README gives the default keys as lowercase (`season_winter`). The
   raid config uses `Season_Winter`. Valheim *probably* lowercases global keys, but that isn't documented.
   Run `globalkeys` on the live server once and match exactly what it prints. `SeasonDay_{n}` was not found in any
   doc; don't build on it until `globalkeys` shows it.
3. **Custom Raids `StopTouchingMyConfigs`.** The wiki says the default is false; the 1.7.10 release notes say
   true. Set it explicitly to `true`, or Custom Raids may rewrite the generated 214-raid file.
4. **Gamma of Night Lights also sets night length.** Seasons and Gamma of Night Lights both control day and
   night length. Let Seasons own it (the Oslo values) and use Gamma of Night Lights for brightness only.

## Top integration moves (ranked)

### Seasons ⇄ raids ⇄ weather (highest value, config-only)
1. **Two-season raids via `RequireOneOfGlobalKeys`**: e.g. wolves/Fenring raids in Fall+Winter, Surtlings in
   Spring+Summer. This makes the rhythm of the year felt without inflating the pool.
2. **Per-spawn season gating** (`RequiredGlobalKey` / `RequiredNotGlobalKey` on spawn entries). One raid can
   field different creatures per season, e.g. a Draugr raid that adds Wraiths only in winter. It doesn't add
   raids to the pool, so the vanilla ratio doesn't move.
3. **Weather-coupled raids**: `ConditionEnvironment` = `Snow, SnowStorm` for winter raids, `Rain, ThunderStorm`
   for fall. Seasons already shifts each biome's weather odds, so raid frequency follows the weather naturally.
   *Check whether Seasons' renamed winter environments match these names; dump the list with
   `WriteEnvironmentDataToDisk=true`.*
4. **Night-only raids scale with the Oslo nights for free**: `CanStartDuringDay=false`. Winter's 72 % night
   gives about 17 minutes of dark per day; summer gives about 6. Watch the `SpawnAtDay=false` despawn behaviour
   and how PersistentRaids interacts with it.
5. **Seasonal trader stock**: `Custom trader items.json` supports `requiredGlobalKey`, so Haldor can sell frost
   mead ingredients or arrows only in winter, ahead of the winter raids.
6. **Stealth/noise**: Seasons already makes players louder on snow. Custom Raids spawns can use
   `ConditionNearbyPlayersNoiseThreshold`, so noisy winter players draw raids.
7. **Freeze the "Deep North/Ashlands later" item**: when you revisit them, Ashlands raids get
   `NotRequiredGlobalKeys=season_winter` and Deep North raids get `season_winter`. Seasons doesn't change those
   biomes' terrain, but this gives a hot-vs-cold contrast.

### Food and farming loop across Seasons + building mods
8. **Winter larder**: Seasons stops crops in winter. OdinsFoodBarrels (50 of one food) + AzuCraftyBoxes (pulls
   from nearby containers) + AutoFeedRedux (animals eat from containers) make "stock up in fall" a real loop.
   Size the feed stock for winter, because animals eat at the normal rate while breeding slows.
9. **One growth knob**: leave PlantEverything growth times at vanilla and tune crops only through Seasons'
   `plantsGrowthMultiplier`. Both mods scale the same timer, so tuning both compounds invisibly.
10. **Fire protects crops**: test whether Balrond/Odin braziers count as a heat source for Seasons' crop
    protection (*inferred*). If they do, the decorative fire pieces gain a gameplay use.
11. **Lightkeeper for winter fuel**: long nights burn more wood (`fireplaceDrainMultiplier`), so Lightkeeper's
    fuel management becomes useful.

### Safety and consistency
12. **AzuCraftyBoxes: exclude every ship** (vanilla + Balrond Shipyard) in `Azumatt.AzuCraftyBoxes.yml`; there is a
    documented item-duplication bug with ship storage. Keep Container Range at 20.
13. **Items added by client-only mods**: ATM's Shovel and BetterArchery's Leather Quiver are items a CORE-pack
    player doesn't have. Stored in a shared chest, they're likely lost on the next save (*inferred*). Either move
    ATM and BetterArchery to CORE, set ATM `Shovel=false`, or tell FULL players not to store those items in shared chests.
14. **SatelliteMap vs Seasons' winter map** (biggest untested risk): SatelliteMap redraws the map, so the winter
    recolour and frozen sea may not show. Test one winter day with `DetailLevel=1`, which is closest to vanilla
    and is set server-wide.
15. **Crater repair**: ATM's reset/raise tools probably repair creature craters (both are standard terrain mods).
    That's fine and makes craters a nuisance rather than permanent. Enable Gizmo's `ignoreTerrainOpPrefab`
    because Gizmo and ATM both use LeftAlt.
16. **GraphicsConfigPlus**: limit it to performance settings (shadows, LOD, vSync). Leave gamma, fog, bloom and
    SSAO at default, so Seasons' seasonal colour and fog read the same for everyone.
17. **Ship identical client configs** in both packs for ATM, QuickStackStore
    (`AllowAreaStackingInMultiplayerWithoutMUC`), BetterArchery and DiscoveryPins. None of them are enforced
    unless installed server-side.

### Trim candidates (to keep it close to vanilla)
18. **OdinsKingdom**: its castle set is largely covered by OdinArchitect and Balrond Constructions. Compare the
    two Odin build menus, then drop it.
19. **Duplicate fish traps**: OdinArchitect's Fish Trap and Balrond Shipyard's Fishnet Trap both do automated
    fishing. Pick one, or disable one recipe.
20. **AAABuildMenu**: vanilla 1.0 has favourites, recents and search. Keep it only for "Can Build" and pinned
    recipes; otherwise it's the most redundant mod in the list.
21. **WhichModAddedThis**: puts mod names on tooltips, which breaks the vanilla feel. Keep it for admins only.
22. **ZenUI**: keep durability bars and biome notifications, and turn off the crafting-panel replacement and
    status-icon hiding. Hiding status icons may hide Seasons' season buff.
23. **Build-menu overflow**: with about 500 building pieces, consider ComfyMods' SearsCatalog. It's the one
    add-on worth it, and it's quality-of-life, not content.

### Classification flags for CORE/FULL
- SatelliteMap and Server_devcommands say server install is optional and clients aren't kicked. Test with a
  CORE client before relying on either being "enforced".
- ATM, DiscoveryPins and QuickStackStore only sync settings if also installed on the server.

## Verify in game, in this order
1. `globalkeys` on the server, for the exact season key strings and whether `SeasonDay_*` exists.
2. Custom Raids `WriteEnvironmentDataToDisk=true`, to see the environment names Seasons actually uses.
3. Set Custom Raids `StopTouchingMyConfigs=true` explicitly.
4. PersistentRaids' own config: find its opt-in duration setting and how it handles leftover raiders across
   46-minute checks.
5. One winter day with SatelliteMap `DetailLevel=1` on a cratered hillside, with ATM and Gizmo loaded.

---

# Appendix A — Seasons, Custom Raids, PersistentRaids, GammaOfNightLights, ConditionalConfigSync, Community Patch

# Integration research: Seasons, Custom Raids, PersistentRaids, Gamma of Night Lights, Conditional Config Sync, Community Patch

**Sourcing limits (read first):**
- thunderstore.io, nexusmods.com and github-wiki-see.page are blocked by the egress proxy here. None of the six Thunderstore pages were read directly, and I could not check the exact pinned versions.
- I used GitHub READMEs and wikis instead, fetched through a summarising fetcher. Quotes below are what the fetcher returned, not guaranteed character-exact.
- They track each repo's current state, not necessarily your pinned versions. Custom Raids 1.8.2 (Sep 10) is confirmed as the latest release, and its Data-Locations file is stamped 1.8.2.
- Gaps: PersistentRaids has no README I could reach. I got only a one-line Thunderstore blurb from search. Gamma of Night Lights config entries are not documented in what I reached. Seasons' per-setting descriptions are thin because the README summaries were lossy. Your in-game `BepInEx/config` tooltips are the authoritative source for those.

Sources:
- https://github.com/shudnal/Seasons and its raw README
- https://github.com/ASharpPen/Valheim.CustomRaids and wiki pages Config-Raids, Config-General, Field-Options, How-Do-Raids-Work, Data-Default-Raids, Data-Locations, Home
- https://github.com/ASharpPen/Valheim.CustomRaids/releases
- https://github.com/shudnal/GammaOfNightLights
- https://github.com/shudnal/ConditionalConfigSync
- https://github.com/MidnightsFX/Valheim-Community-Patch

---

## 0. Correction to your setup assumptions

The Seasons README says the default keys are lowercase, `season_spring`, `season_summer`, `season_fall` and `season_winter`:

> "You can enable setting of season related server wide global key. It's disabled by default. You can customize the key in case you need it."

- **Case:** you wrote `Season_Spring`. I believe Valheim lowercases global keys on set and get, so case should not matter. That is my inference, not documented. Write `season_winter` in configs to be safe.
- **`SeasonDay_{n}` is not in anything I could read.** Verify it exists by running Custom Raids with `WriteGlobalKeyDataToDisk=true`, or by using the `globalkeys` console command on a live server. Do not build raids on it until confirmed.
- The key names are customisable. That is the documented escape hatch if a key collides with another mod.

## 1. Seasons: inventory

### Config files
All are JSON in `BepInEx/config/shudnal.Seasons/`:
- `Spring.json`, `Summer.json`, `Fall.json`, `Winter.json`
- `Custom environments.json`
- `Custom events.json`
- `Custom lightings.json`
- `Custom stats.json`
- `Custom trader items.json`
- `Seasonal snow.json`
- `Custom grass settings.json`
- `Custom clutter settings.json`
- `Custom biome settings.json`
- `Custom world settings.json`

### Per-season scalar settings
Names as listed in the README. Defaults were only given for the first two.

| Setting | Effect |
|---|---|
| `daysInSeason` | Season length. Default 10; you set 91. |
| `nightLength` | Night as a percentage of the day. Default 30; you have it per season. |
| `plantsGrowthMultiplier` | Crop growth speed. README: "Plants grow slower in fall and stop in winter but grow quickly in spring and summer." |
| `beehiveProductionMultiplier` | Honey rate. |
| `foodDrainMultiplier` | Food drain speed. |
| `staminaDrainMultiplier` | Stamina use. |
| `fireplaceDrainMultiplier` | Wood burn rate of fires. |
| `sapCollectingSpeedMultiplier` | Sap collection. |
| `rainProtection` | "Prevents building pieces from weather damage" in the listed seasons. |
| `woodFromTreesMultiplier` | Wood drops from trees. |
| `windIntensityMultiplier` | Wind strength. |
| `restedBuffDurationMultiplier` | Rested buff duration. |
| `livestockProcreationMultiplier` | "breeding speed and conditions for creatures". |
| `meatFromAnimalsMultiplier` | Meat drops from animals. |
| `treesRegrowthChance` | Sapling generation probability. |
| `torchAsFiresource` | Torches give warmth but durability drain rises (quoted 0.0333 to 0.1 while providing warmth). |
| `overheatIn2WarmClothes` | Summer "Warm" effect from wearing two frost-resistant pieces; it cuts stamina and eitr regen by 20%. |

Pickable resource growth is also configurable per season, but I found no setting name.

### Weather and environments (`Custom environments.json`)
- Per-environment properties: `m_isWet`, `m_isFreezing`, `m_isFreezingAtNight`, `m_isCold`, `m_isColdAtNight`, `m_alwaysDark`, `m_snowBuildup`, `m_auroraColors`, cloud opacity by time of day, AO, ambient loop, particle systems.
- Biome environment tables: per-biome weather distribution by environment name and weight.
- This is how you control the wet/cold/freezing debuffs at the weather level. The README says the freezing debuff applies "in mountains/deep north during cold seasons", cold applies in fall and winter, and wet applies in rain.
- Documented safety note: "You won't die from freezing debuff if you're not in the mountains or deep north. You will stay at very low hp."

### Raids and events (`Custom events.json`)
- Fields are `m_name`, `m_biomes` (comma list) and `m_weight` (0 = never, 1 = normal).
- Default winter: no skeletons, blobs, trolls or surtlings; more dragons and wolves. Default summer: no wolves. Default fall: draugr and moisture-themed.
- Not documented: whether an event `m_name` that only exists as a Custom Raids raid can be reweighted. Seasons edits the vanilla `RandEventSystem` list; Custom Raids adds or overrides entries in the same list. So the name match probably works, but that is inference and worth a test.

### Global keys
- Seasonal keys: default off, customisable (see section 0).
- Trader items (`Custom trader items.json`) carry a `requiredGlobalKey` field. Haldor and Hildir are supported, each item with `prefab`, `stack` and `price`.

### Lighting and night (`Custom lightings.json`)
- Per time of day (morning, day, evening, night): `luminanceMultiplier`, `fogDensityMultiplier`, `lightIntensityDayMultiplier` and `lightIntensityNightMultiplier`, `screenColorTemperature`.
- `nightLength` is in Seasons and also in Gamma of Night Lights (see section 4).

### Character stats (`Custom stats.json`)
- Per-season status effect with health and stamina regen multipliers, damage modifiers for every damage type, speed, stealth and noise modifiers, skill changes, fall damage.
- Stealth and noise modifiers interact with Custom Raids' `ConditionNearbyPlayersNoiseThreshold` (see section 3).

### World and map
- `Custom biome settings.json`: per-biome seasonal terrain colours. Ashlands, Mountain and Deep North do not change terrain.
- Winter minimap colours per biome, with a default that interpolates toward `#FAFAFF`. The README says a season transition is required to apply changes.
- `Custom grass settings.json`: patch size and density per season.
- `Custom clutter settings.json`: seasonal flower prefabs. Three new ones ship: Meadows red/blue, Black Forest pink, Swamp white.
- `Seasonal snow.json`: per-piece `buildup` of Seasonal, Reduced, Ignore or Disabled. Also creature and cape snow materials.
- `Custom textures` and `Custom music` folders.
- `Custom world settings.json`: real-time seasons, with a UTC start time and day length in seconds. The season changes when the time comes, and this prevents manual season change by sleeping.

### Off by default but commonly recommended
- **The seasonal global key toggle.** You have already turned it on.
- **Real-time seasons** (`Custom world settings.json`). This is an example entry only. The README says seasons otherwise change on the morning of the first day and can be set to change only when sleeping.
- **`torchAsFiresource`, `overheatIn2WarmClothes` and `rainProtection`.** I could not confirm their defaults from what I read. Check them.
- I found no README claim of "commonly recommended" for any setting beyond this.

## 2. Seasons' server behaviour

- It must be installed on the server and on every client.
- Settings marked `[Synced with Server]` sync by default, and world state, physics and progression are always server-controlled.
- Sync policy lives in `ConditionalConfigSync.SyncPolicy.cfg`, where you prefix a setting with `+` (server-controlled) or `-` (client-controlled).
- Visual files (textures, music, lighting) can be client-customised.
- The README lists no known conflicts.
- Steam launch options `-gfx-enable-gfx-jobs -gfx-enable-native-gfx-jobs` are suggested for FPS.
- RenderLimits "Distance area" above 10 costs FPS in forests.

## 3. Custom Raids (source: wiki Config-Raids, Config-General, Field-Options)

### General config
- `[General]`: `LoadSupplementalRaids` (true), `GeneratePresetRaids` (true), `StopTouchingMyConfigs` (false; the default became true in 1.7.10 per the release notes, so the wiki and release notes disagree), `PauseEventTimersWhileOffline` (true).
- `[EventSystem]`: `RemoveAllExistingRaids` (false), `OverrideExisting` (true, meaning same-name events are overridden), `EventCheckInterval` (46 min), `EventTriggerChance` (20).
- `[IndividualRaids]`: `UseIndividualRaidChecks` (false; "Allows individual frequencies and chances per raid") and `MinimumTimeBetweenRaids` (46).
- `[Debug]`: `WriteEnvironmentDataToDisk`, `WriteGlobalKeyDataToDisk`, `WriteLocationsToDisk`, `WriteDefaultEventDataToDisk`, `WritePostChangeEventDataToDisk`, `DebugOn`, `TraceLogging`, `DebugFileFolder`.

### Raid-level fields
**Identity and timing**
- `Name` (same name overrides an existing raid), `Enabled`, `Duration` (90 s default), `StartMessage`, `EndMessage`.
- `Random`, `RaidFrequency` (46 min), `RaidChance` (default 0).
- `OnStopStartRaid` (chain to another raid), `PauseIfNoPlayerInArea`, `NearBaseOnly`.

**Where and when**
- `Biomes`, `CanStartDuringDay`, `CanStartDuringNight`.
- `ConditionAltitudeMin`/`Max` (distance to water surface), `ConditionDistanceToCenterMin`/`Max`.
- `ConditionWorldAgeDaysMin`/`Max`.

**Global keys**
- `RequiredGlobalKeys`: all must be present.
- `NotRequiredGlobalKeys`: any present disables the raid.
- `RequireOneOfGlobalKeys`: at least one required.

**Environment and presentation**
- `ConditionEnvironment` (list of environments that enable the raid).
- `ForceEnvironment` (sets weather during the raid), `ForceMusic`.

**Location and proximity**
- `ConditionLocation` (list of location prefab names where the raid is enabled, for example `Runestone_Boars, StartTemple`).
- `ConditionMustBeNearPrefab`, `ConditionMustBeNearAllPrefabs` and `ConditionMustNotBeNearPrefab`, each with a `...Distance` field (default 100).

**Players**
- `ConditionPlayersNearbyMin`/`Max`, `ConditionPlayersOnlineMin`/`Max`.
- `ConditionPlayerMustHaveAnyOfPlayerKeys`, `ConditionPlayerMustNotHaveAnyOfPlayerKeys`, `ConditionPlayerMustHaveAllOfPlayerKeys`.
- `ConditionPlayerMustKnowAnyOfItems`, `ConditionPlayerMustNotKnowAnyOfItems`.

**Faction**
- `Faction`: "Assign a single faction to all entities in raid". Default Boss.

### Spawn-level fields
- **Identity and count:** `Name`, `Enabled`, `PrefabName`, `MaxSpawned`, `GroupSizeMin`/`Max`, `GroupRadius`.
- **Timing and chance:** `SpawnInterval`, `SpawnChancePerInterval`, `SpawnAtNight`, `SpawnAtDay`. `SpawnAtDay=false` can cause despawning.
- **Position:** `SpawnDistance`, `SpawnRadiusMin` (40) and `SpawnRadiusMax` (80), `GroundOffset`.
- **Behaviour and level:** `HuntPlayer`, `MinLevel`/`MaxLevel` (2 = one star).
- **Keys and weather:** `RequiredGlobalKey`, `RequiredNotGlobalKey`, `RequiredEnvironments`.
- **Terrain:** `AltitudeMin`/`Max`, `TerrainTiltMin`/`Max`, `InForest`, `OutsideForest`, `OceanDepthMin`/`Max`, `InLava`, `OutsideLava`, `InsidePlayerBase`.
- **Faction and range:** `Faction` (per mob), `ConditionDistanceToCenterMin`/`Max`, `ConditionWorldAgeDaysMin`/`Max`, `DistanceToTriggerPlayerConditions` (100).
- **Player-carried conditions:** `ConditionNearbyPlayersCarryValue`, `ConditionNearbyPlayerCarriesItem`, `ConditionNearbyPlayersNoiseThreshold`.

### Exact accepted strings (Field-Options)
- **Factions:** `Players, AnimalsVeg, ForestMonsters, Undead, Demon, MountainMonsters, SeaMonsters, PlainsMonsters, Boss, MistlandsMonsters, Dverger, PlayerSpawned, TrainingDummy, DeepNorth`.
- **Biomes:** `Meadows, Swamp, Mountain, BlackForest, Plains, AshLands, DeepNorth, Ocean, Mistlands`.
- **Environments:** `Clear, Twilight_Clear, Misty, Darklands_dark, Heath clear, DeepForest Mist, GDKing, Rain, LightRain, ThunderStorm, Eikthyr, GoblinKing, nofogts, SwampRain, Bonemass, Snow, Twilight_Snow, Twilight_SnowStorm, SnowStorm, Moder, Ashrain, Crypt, SunkenCrypt`.
  - That list is what the wiki shows; it may be incomplete. The Data-Default-Raids page also uses `CombatEventL2`, `CombatEventL3` and `CombatEventL4` as `ForceEnvironment` values, and `Ghosts`, `Queen` and `Fader`. Dump the real list with `WriteEnvironmentDataToDisk=true`.
  - Seasons' custom environments are not listed. Whether `ConditionEnvironment` can match a Seasons-added environment is undocumented.
- **Global keys in the wiki list:** defeated_eikthyr, gdking, bonemass, dragon, goblinking, queen, hive, plus KilledTroll, killed_surtling, KilledBat and Hildir keys.

### Semantics the docs do not pin down
- **`RaidChance` versus `EventTriggerChance`.** Docs say `RaidChance` is the "Chance at each check for this raid to run" (default 0) and `RaidFrequency` is "Minutes between checks for this raid to run". The wiki only ties it to `UseIndividualRaidChecks` ("individual frequencies and chances per raid").
  - Presumably with that flag off, the global 46 min / 20% check picks one raid from the eligible pool, which matches your "one raid, equal weight" description. With it on, each raid rolls independently.
  - I found no statement on whether the two modes can coexist, or whether independent rolls can fire several raids at once. Flagged as inference.
- **Raid-level `ConditionEnvironment` versus `ForceEnvironment`.** The first is checked against the current weather, the second is applied on raid start. Both exist per the wiki. The release notes say `ConditionEnvironment` was added in v1.5.0.
- **Client spawn behaviour.** "For each mob they will check if spawn conditions are right. THIS is what usually makes most raids stumble" (How-Do-Raids-Work). Raid start is decided by the host, but spawning is done by clients in their area, so spawn-level conditions (`RequiredEnvironments`, altitude, forest and so on) can silently suppress a raid that started.

### PersistentRaids interaction
The search blurb says it "makes raid creatures hunt and persist after the HUD [sic]. Open-world mobs stay vanilla, and per-raid Custom Raids durations remain untouched unless you opt in."
- From that I infer it is aware of Custom Raids and leaves `Duration` alone by default.
- The opt-in setting name, and how it affects `SpawnAtDay=false` despawning or `HuntPlayer`, are not documented where I could read.
- With 214 raids and a 46-minute check, leftover persistent raiders will stack across raid starts. Test this on the server.

## 4. Other mods

**Gamma of Night Lights** (GitHub README only)
- Controls night luminance, fog, moon and sun intensity, plus "night-to-day cycles without affecting overall day length". Day length is in seconds; it says vanilla is 1800, though your setup is 1440.
- It is server-synced via Conditional Config Sync.
- **Overlap risk with Seasons.** Both set night length and lighting. The README does not say which wins. Pick one owner for day length and night length, and use the other only for visuals.

**Conditional Config Sync**
- Shared sync library that does not interfere with ServerSync or Jotunn.
- Policy files are in `BepInEx/config/shudnal.ConditionalConfigSync/` and can be edited while the server runs.
- Three policy types control sync, mod requirements and config-manager visibility.
- It rejects embedded copies, so it must be a standalone dependency.

**Valheim Community Patch**
- Vanilla bug and performance fixes only. Its README "does not mention raids, events, global keys, or weather".
- Needs BepInEx and Jotunn. Server and client need matching major.minor versions. Its config syncs only to clients that have the mod.

## 5. Integration ideas

**Documented:**
1. **Season-gated raids (you already use this).**
   - `RequiredGlobalKeys=season_winter` on a custom raid, or `NotRequiredGlobalKeys=season_summer`.
   - `RequireOneOfGlobalKeys=season_fall,season_winter` for two-season raids. You can use one raid for that instead of cloning.
   - The same works at spawn level via `RequiredGlobalKey` and `RequiredNotGlobalKey`, so one raid can field different mobs per season.
2. **Gate the ungated Deep North and Ashlands raids.** Ashlands and Deep North do not change terrain in Seasons. Use `season_winter` on Deep North and `NotRequiredGlobalKeys=season_winter` on Ashlands if you want a hot-and-cold feel. Seasons' own freezing debuff already targets mountain and Deep North in cold seasons.
3. **Rebalance the vanilla share with `Custom events.json`.** Seasons weights vanilla raids per season (you already do this). The fix for a 39-42% vanilla share is either zeroing more vanilla `m_weight` values, or raising `EventTriggerChance`/`RaidChance`. There is no pool-weight field in Custom Raids.
4. **`UseIndividualRaidChecks=true` with per-season `RaidChance`.** Clones of a raid in each season can carry different `RaidFrequency` and `RaidChance`, so you tune threat per season without touching the equal-weight pool. Caveat: semantics undocumented (see section 3).
5. **`ConditionEnvironment` weather-coupled raids.** Raids that need `SnowStorm` or `Snow` in winter and `ThunderStorm` or `Rain` in fall. Seasons shifts the biome weather tables, so combined with `ForceEnvironment` and `ConditionEnvironment` the weather and raid frequency move together.
6. **Night-coupled raids.**
   - `CanStartDuringNight=true` and `CanStartDuringDay=false` with `SpawnAtDay=false`.
   - Winter's 72% night gives about 17 minutes of dark per 24-minute day, so night-only raids have a large window in winter and a small one in summer. That is an emergent seasonal scaling for free.
   - Keep the `SpawnAtDay=false` despawn warning in mind, and PersistentRaids may keep them alive past dawn.
7. **Player-progression gating.** `ConditionPlayerMustHaveAnyOfPlayerKeys`, `ConditionWorldAgeDaysMin`/`Max`, `ConditionPlayersOnlineMin`. With 5 players, use `ConditionPlayersNearbyMin=3` for heavy raids.
8. **Location-specific raids.** `ConditionLocation` with the prefabs in Data-Locations (for example runestones, a StartTemple, per-biome crypts), combined with a season key: a winter ambush at a particular runestone.
9. **Stealth and noise.** Seasons' stats can change `m_noiseModifier` and `m_stealthModifier`. Raid spawns can use `ConditionNearbyPlayersNoiseThreshold`. Winter stealth penalty plus a noise-triggered spawn gives a consistent theme. The values need testing.
10. **Seasonal trader unlocks.** `requiredGlobalKey` in `Custom trader items.json` can use the season key, so a winter-only Haldor item fits the winter raids. It is also the route to "supplies appear before the raid season".
11. **One sync policy.** All of Seasons, Gamma of Night Lights and Custom Raids' client sync run through the same library, so one `SyncPolicy.cfg` can make a single ruleset, and lock visual options per player if the owner wants.
12. **Faction-based raid themes.** `Faction=ForestMonsters` and `Undead` make raid creatures friendly to their faction mates. `Faction=Players` would presumably make them neutral to players, but that is undocumented and risky.

**Inference only (not in docs):**
- Reweighting custom raid names in Seasons' `Custom events.json` (see section 1).
- Keys such as `SeasonDay_{n}` for time-within-season gating, if they exist. If they do, `ConditionWorldAgeDaysMin` is not needed for that.
- Seasons' custom environments as valid `ConditionEnvironment` names.
- Independent-check semantics when `UseIndividualRaidChecks=true`.
- Using `OnStopStartRaid` to chain the season's two raids into a two-stage event.

## 6. Conflicts, warnings and dedicated-server notes

- **Key case and name mismatch** (section 0). Confirm the published key strings before writing config.
- **Night and day length are controlled in two places** (Seasons and Gamma of Night Lights). Set them in only one.
- **The raid pool is one shared roll.** Custom Raids has no per-raid weight field. More clones of a raid raise its frequency; gating does not change that.
- **Raid spawning is client-side** (How-Do-Raids-Work). Spawn conditions are evaluated by clients near the raid, so mismatched `RequiredEnvironments` on spawns, or a `RequiredGlobalKey` a client has not received, can suppress a raid that started.
- **Custom Raids must be on all clients and the server.** Since 1.2.0 clients request server configs automatically. A client with different config files gets the server's for that session.
- **`PauseEventTimersWhileOffline`** defaults to true, which matters for a private server that is often empty.
- **`StopTouchingMyConfigs`** default conflicts between the wiki (false) and release notes (true since 1.7.10). Set it explicitly, or Custom Raids may rewrite your 214-raid config.
- **Seasons cannot be changed by sleeping when real-time seasons are on.** With your current setup (91-day seasons), seasons change on the morning of day 1 of the new season.
- **Changed Seasons visuals** (map colours, textures) only apply after a season transition.
- **Custom Raids 1.8.0** added World Advancement Progression support. Not relevant unless you use that mod, but it means the private-keys lookup is server-side.
- **Community Patch** adds no known conflict with the others, and it is safe if only the server has it.
- **Nothing I could read documents any direct conflict** between Seasons, Custom Raids, PersistentRaids, Gamma of Night Lights and Conditional Config Sync. That is absence of evidence, not clearance, given that I could not read PersistentRaids beyond a one-line description.

## 7. What I would verify in-game before building more
1. `globalkeys` output on the live server, to confirm the exact season key names and whether `SeasonDay_*` exists.
2. The environment list from `WriteEnvironmentDataToDisk=true`, to see Seasons' added environments.
3. Whether `UseIndividualRaidChecks=true` still respects the vanilla-share pool, using `TraceLogging` briefly.
4. PersistentRaids' own config file, for its opt-in duration setting and whether it touches `HuntPlayer` or despawn behaviour.

---

# Appendix B — Building and content mods

# Valheim server mod integration report

## Research coverage

Thunderstore, old/new/valheim.thunderstore.io, Nexus and hexium.gg are blocked by the egress proxy. WebFetch returned `EGRESS_BLOCKED` for all of them.

What I could read:
- GitHub READMEs: Vapok/AutoFeedRedux, shudnal/ExtraSlots, shudnal/Seasons, and OrianaVenture/Valheim-MissingPieces.
- WebSearch snippets of the Thunderstore pages, which are the descriptions, not full READMEs.

Most of it is at description level. I could not read piece-by-piece lists or full config tables for the Balrond mods, OdinPlus mods, PlantEverything, AzuCraftyBoxes, and zukane's two mods. Per-piece overlaps below are therefore inferred from descriptions and labelled that way. Please check the real `.cfg` files and in-game build menus before acting.

## 1. Overlap and redundancy, ranked by drop confidence

| Overlap | Evidence | Verdict |
|---|---|---|
| **Fish trap: OdinArchitect vs Balrond Shipyard.** | OdinArchitect has a "Fish Trap (Smelter: Place worms to receive raw fish)", plus a Bird House and a Compost that turns raw meat into worms. Balrond Shipyard has a "Fishnet Trap... catch biome-appropriate fish more effectively when supplied with the correct bait." | Documented duplicate function. Two automated fishing structures is the clearest redundancy in the list. |
| **Castle/stone building: OdinsKingdom vs OdinArchitect vs Balrond Constructions vs MissingPieces.** | OdinsKingdom adds round tower walls and floors, 1x1 stone walls, castle round stairs, a drawbridge, doors, a trapdoor hatch and beam decor. OdinArchitect adds 205+ pieces including functional drawbridges, hidden hatches, gates, elevators and Dvergr marble. Balrond Constructions adds 150+ pieces including gates, platforms, trims, stairs, secret doors and arches in stone, marble, clay, darkwood, hardwood, crystal and iron. MissingPieces adds stone stair corner, 1 m stone floor, stone ramps and triangular stone walls. | Documented overlap in category: drawbridges, hatches/secret doors, gates, stairs, and stone walls/floors. I did not see per-piece names for Constructions, so exact duplicates are not confirmed. **Candidate to drop: OdinsKingdom.** It is the smallest and oldest, and its castle set is largely covered by OdinArchitect's. Decide after comparing the two build menus. |
| **Storage pieces: MissingPieces vs Balrond Furniture Reborn vs OdinsFoodBarrels.** | MissingPieces adds `piece_chest_wooden_drawer`. Furniture Reborn adds usable chests and storage crates, for example `piece_chest_private_bal` and `dvergr_chest_bal`. OdinsFoodBarrels adds baskets and barrels that hold 50 of one food type. | Drawer and chest overlap is cosmetic. OdinsFoodBarrels is a different function because it is dynamic single-type storage. Keep it only if the crew uses it (see section 3). |
| **Ladders and stairs.** | MissingPieces adds a finer step-ladder and a narrow wood stair. Furniture Reborn has climbable ladders. OdinsKingdom has a small wooden ladder and castle round stairs. | Near-certain partial overlap. Low value to chase. |
| **Hearth and ward-like items: Hearth Marks and Lightkeeper.** | Hearth Marks adds a piece that names the zone around you. Lightkeeper is a "ward-like control for lights and fuel management". | Different functions. Hearth Marks is the cosmetic/utility piece, and it is the smaller and less essential of the two. |
| **Banners and carpets: Furniture Reborn vs BannerColorizer.** | Furniture Reborn adds rugs, curtains and banners, and BannerColorizer dyes "roof, banner, carpet, and crystal wall". | Complementary, not duplicates. BannerColorizer only adds value if the pieces it dyes are in use. |
| **Weapon Crate.** | It turns weapons and armor into decorative build pieces, like the vanilla serving tray. | Unique. Note it is content-adjacent: it also turns armor into placeable pieces, which sits close to the armor theme the owner removed. |
| **Piece-ID collision risk.** | A search surfaced a Missing Pieces conflict with OCDheim where pieces disappeared. The same collision class could apply between building mods. | Undocumented for this mod set. |

**Bottom line:** if the owner wants to drop one mod, OdinsKingdom is the best candidate, with Hearth Marks as a lower-cost second.

## 2. Config settings worth tuning

### AzuCraftyBoxes
Sources: [Thunderstore search snippet](https://thunderstore.io/c/valheim/p/Azumatt/AzuCraftyBoxes/), [LongshipUpgrades issue #4](https://github.com/shudnal/LongshipUpgrades/issues/4).
- **Container Range.** This is synced with the server. The default is 20. Crafting and building pull from every container within that radius, including Furniture Reborn chests and Food Barrels. 20 is fine for a base. I would not raise it, because a larger radius makes crafting absorb unrelated stock.
- **Leave One Item.** This is synced with the server. The documented purpose is to leave one of each item in the container.
- **`Azumatt.AzuCraftyBoxes.yml`.** It holds per-container exclusion lists. The documented example is `VikingShip: exclude: - All`. **Exclude every ship, and Shipyard's custom ships**, because the LongshipUpgrades report describes item duplication when crafting from upgraded ship storage. Ship exclusion for Balrond's Knarr, Holk and Snekke is my inference. Test that it works.
- **Non-default vs default ranges.** The yml exclusion format is documented only by example, so check the generated file for the real schema.

### PlantEverything
Sources: [changelog](https://thunderstore.io/c/valheim/p/Advize/PlantEverything/changelog/), [Nexus page](https://www.nexusmods.com/valheim/mods/1042).
- **Documented Seasons compatibility.** Version 1.20.0 "added Plant and Pickable growth timer compatibility with Seasons by shudnal". From the wording this is the growth timer hover text. I could not confirm whether it is more than that.
- **`[General] LockConfiguration`.** Set it to true so only admins can edit config.
- **`[General] EnableExtraResources`.** It lets you define any prefab (even from other mods) to add to the cultivator build table. Keep it off unless there is a specific gap.
- **`[Difficulty] RecoverResources`.** It is off by default.
- **Growth time, cost, yield and grow radius.** These are documented as configurable per crop, sapling and pickable. I could not read the setting names.
- **Seasons interaction.** Seasons has a `plantsGrowthMultiplier` for plants and pickables. **Keep PlantEverything growth time at vanilla ratios and tune only with Seasons' multiplier.** Both mods scale the same timer, so stacking them can silently make crops grow very slowly or very fast. This is an inference, but I consider it high-value.
- **Winter and dying crops.** I found no documentation of PlantEverything interacting with Seasons' winter die-off or ground reversion, and the Seasons README I read does not mention it either.

### AutoFeedRedux
Source: [README](https://github.com/Vapok/AutoFeedRedux). The file is `BepInEx/config/vapok.mods.AutoFeedRedux.cfg`.

| Setting | Default | Notes |
|---|---|---|
| Enable Auto Feeder | true | |
| Feed Range (Meters) | 30.0 | A search snippet said 10 m, so the README and snippet disagree. Check the generated config. |
| Require Move to Feed | true | Animals walk to the container. Keep it true, because it looks more like vanilla. |
| Move Proximity | 1.0 | |
| Protect Containers | true | Stops taming-stage wild creatures from attacking food containers. |
| Disallow Feed | empty | Comma-separated item names. |
| Disallow Animal | empty | Comma-separated creature names. |

- **Interaction with Seasons.** I found no documentation of AutoFeedRedux reading Seasons' `livestockProcreationMultiplier`. The multiplier "affects breeding speed and conditions for creatures" (Seasons README). In winter, breeding slows, but tames still eat. Food consumption by tames is unchanged, so **the food stockpile has to be sized for winter, not spring**. That is inference.
- **Feed Range against the pen.** Set the range to cover the pen only, so tames do not wander to a distant chest.
- **Blacklisting the crops.** `Disallow Feed` could block a crop such as `Carrot, Turnip` so tames do not drain the winter seed stock. Seeds are consumed only if the animal accepts them, so check what each creature eats.
- **Server sync.** The README says the mod "enforces server-side configuration synchronization" on dedicated servers.

### ExtraSlots
Source: [README](https://github.com/shudnal/ExtraSlots).
- **Synced by default.** Slot counts, availability, progression, item eligibility, weight factors and death rules are `[Synced with Server]`. UI layout, hotkeys and labels stay client-side.
- **Row adjustment.** The visible inventory is the native row count plus the configured ExtraSlots row adjustment, with at least one regular row. **Leave the row adjustment at 0** so inventory stays vanilla-sized. ExtraSlots adds dedicated equipment, food, ammo and misc slots plus quick slots. Turn off slot types the crew does not want.
- **Defaults.** The README does not list them. Read `shudnal.ExtraSlots.cfg`.
- **Not everyone needs it.** The page says not every player must have it installed to join, but players without it cannot see the extra slot UI. On a 5-player private server, install it for everyone.
- **Dependency.** It requires ConditionalConfigSync, which Seasons also needs.

### zukane
Source: [Thunderstore page](https://thunderstore.io/c/valheim/p/zukane/Adjustable_Breeding_Limit/).
- **Adjustable Breeding Limit.** It has one setting, the max number of tamed animals allowed before breeding stops. The config file is `zukane2.AdjustBreedingLimit.cfg`.
- **Adjustable Taming Speed 2.** It has one setting, the taming speed. The file is `zukane2.AdjustTamingSpeed.cfg`.
- **Interaction with Seasons.** Both mods act on the same breeding and taming rules Seasons modifies. I found no documented conflict, and I infer Seasons' multiplier scales speed while zukane's caps count. Check that they do not double-penalise in winter.
- **Server sync.** I could not confirm whether they are server-synced, so assume the server config is authoritative and verify it.

## 3. Integration ideas

1. **Seasons-driven food economy (inference).** OdinsFoodBarrels stores 50 of one seed, fruit or vegetable, acting like a stone pile. Combined with AzuCraftyBoxes, the crew can stockpile autumn harvest and have AutoFeedRedux tames eat from it all winter, when plants do not grow. This uses existing pieces, with no new mod.
2. **One storage cluster per base (inference).** Put the Food Barrels and Furniture Reborn chests within AzuCraftyBoxes' 20 m range of the workbench. Keep the feed trough chest inside AutoFeedRedux's range of the pen but outside the crafting range if you want to protect animal feed.
3. **Shipyard plus CraftyBoxes.** Exclude ships in the yml (section 2).
4. **Lightkeeper for winter.** Seasons makes fire protect crops, and winter is long, so a fuel-management piece is useful. Lightkeeper controls lights and fuel. This is a fit, not a documented integration.
5. **Crop protection with existing fire pieces (documented in your brief).** Fire protects crops in Seasons. The decorative braziers and hearths in Furniture Reborn or OdinsKingdom could serve as protection. I found no documentation that modded fire pieces count as a protective heat source, so this is inference and worth testing in game.
6. **Weapon Crate as a trophy room.** Use it for display only. It is the lightest way to give gear a home without adding a mod.
7. **Build-menu overflow.** The MissingPieces page warns of "build menu space limitations due to vanilla pagination" and recommends SearsCatalog by ComfyMods. With about 500 pieces across these mods, this is a real gap. SearsCatalog is the one add I would consider, and it is a quality-of-life fix, not content.

## 4. Conflicts, load order and dedicated-server notes

- **Everything is client and server.** Seasons requires installation on both server and all clients. Balrond WeaponCrate is stated as "required to be both present on Server and all Clients". Treat all piece mods the same way.
- **ConditionalConfigSync.** Seasons requires it, and ExtraSlots needs it too.
- **Drop That.** Seasons PR #47 fixed Drop That compatibility in seasonal meat drops ([PR #47](https://github.com/shudnal/Seasons/pull/47)). If you run Drop That, it interacts with Seasons' meat yields.
- **Seasons version and game version.** Seasons 1.8.2 needed fixes for Valheim 1.0.7 and was tested on Steam 1.0.12 ([PR #43](https://github.com/shudnal/Seasons/pull/43), [issue #44](https://github.com/shudnal/Seasons/issues/44)). Confirm your Seasons version matches the game build.
- **Ship storage dupe.** See the AzuCraftyBoxes yml exclusion in section 2.
- **OdinArchitect and Valheim+.** A search snippet says it is not compatible with Valheim+. Only relevant if you run that.
- **Piece-ID collisions.** The Missing Pieces / OCDheim conflict ([issue #20](https://github.com/BentoGambin/Valheim-MissingPieces/issues/20)) shows this failure class. Nothing in these mods is documented to collide, but two building mods with overlapping stone pieces are the place to look if pieces vanish.
- **Load order.** I found no documented load-order requirements for any of these mods.

## Not verified, and what to do about it

- Exact piece lists for Constructions, Furniture Reborn, OdinArchitect and OdinsKingdom. Compare the two OdinPlus build menus in game.
- Config defaults for ExtraSlots, PlantEverything, AzuCraftyBoxes beyond Container Range, and the zukane mods. Open the generated `.cfg` files.
- Whether PlantEverything interacts with Seasons' crop die-off. I found no documentation either way.

Sources:
- [AzuCraftyBoxes](https://thunderstore.io/c/valheim/p/Azumatt/AzuCraftyBoxes/)
- [AutoFeedRedux](https://github.com/Vapok/AutoFeedRedux)
- [ExtraSlots](https://github.com/shudnal/ExtraSlots)
- [Seasons](https://github.com/shudnal/Seasons)
- [PlantEverything changelog](https://thunderstore.io/c/valheim/p/Advize/PlantEverything/changelog/)
- [MissingPieces (maintained fork)](https://github.com/OrianaVenture/Valheim-MissingPieces)
- [balrond constructions](https://thunderstore.io/c/valheim/p/Balrond/balrond_constructions/)
- [balrond furniture reborn](https://thunderstore.io/c/valheim/p/Balrond/balrond_furniture_reborn/)
- [balrond shipyard](https://thunderstore.io/c/valheim/p/Balrond/balrond_shipyard/)
- [balrond WeaponCrate](https://thunderstore.io/c/valheim/p/Balrond/balrond_WeaponCrate/)
- [OdinArchitect](https://thunderstore.io/c/valheim/p/OdinPlus/OdinArchitect/)
- [OdinsKingdom](https://thunderstore.io/c/valheim/p/OdinPlus/OdinsKingdom/)
- [OdinsFoodBarrels](https://thunderstore.io/c/valheim/p/OdinPlus/OdinsFoodBarrels/)
- [zukane mods](https://thunderstore.io/c/valheim/p/zukane/)

---

# Appendix C — Client-side and QoL mods

# Research report: Valheim client-side mod audit (5-player private server)

**Source limits.** The egress proxy blocks thunderstore.io, nexusmods.com, steamcommunity.com and hexium.gg, so I could not open any Thunderstore page directly. What I used:
- Raw GitHub READMEs, which Thunderstore pages usually mirror. Read in full: AdvancedTerrainModifiers (repo name `searica/TerrainTools`), DiscoveryPins, ComfyGizmo, ComfyAutoRepair, MassFarming, WhichModAddedThis, BuildingHealthDisplay, GraphicsConfigPlus, QuickStackStore and Seasons.
- WebSearch result text that quotes Thunderstore and Nexus pages. This covers SatelliteMap, Server_devcommands, AAABuildMenu, ZenUI, UnRemove and BetterArchery.

Where a statement rests only on search-engine paraphrase I mark it **[snippet]**. Where it is my reasoning rather than documentation I mark it **[INFERRED]**.

Two version caveats: the DiscoveryPins README predates the listed 0.4.1, and the BetterArchery config list I found is from v1.7.6, not 2.0.2.

## 1. Client/server classification

| Mod | Verdict | Evidence |
|---|---|---|
| ATM 1.5.4 | Client-only works. Has item side effects (see below). | README: "This mod does work as a client-side only mod and only needs to be installed on the server if you wish to enforce configuration settings." https://github.com/searica/TerrainTools |
| DiscoveryPins 0.4.1 | Client-only works. Optional server sync. | "Can be used as a purely client side mod or it can be installed on the server to sync some of the settings." Synced: "whether the keyboard shortcut is enabled and how far away it can trigger automatic pins." https://github.com/searica/DiscoveryPins |
| Gizmo 1.16.0 | Client. | Thunderstore category "Client-side Utility Building" [snippet]. README has no server section. https://github.com/BruceOfTheBow/BruceComfyMods/tree/main/ComfyGizmo |
| ComfyAutoRepair 1.1.0 | Client. | Tagged "client-side tweak" [snippet]. README describes only local behaviour. |
| QuickStackStore 1.4.15 | Client. Optional server install syncs some settings. | "designed to be a client side mod. If you optionally also install it on the server, you get the option to server sync all the area stacking and restocking config settings from section 2." [snippet, https://thunderstore.io/c/valheim/p/Goldenrevolver/Quick_Stack_Store_Sort_Trash_Restock/] |
| MassFarming 1.13.0 | Client. | Categorised "Client-side" [snippet]. README describes only local actions. https://github.com/Xeio/MassFarming |
| WhichModAddedThis 0.1.2 | Client. | README: "This is a client side mod." Needs BepInEx and Jotunn. https://github.com/MSchmoecker/WhichModAddedThis |
| UnRemove 1.0.0 | Client. | "client-side mod in the Tweaks/Misc category" [snippet, https://thunderstore.io/c/valheim/p/Neobotics/UnRemove/]. |
| ZenUI 1.8.11 (+ Zen_ModLib) | Client. | "categorized as a client-side mod" [snippet]. A changelog note says "Enable Crafting Panel ... no longer requires admin status and does not need to be installed on the server" [snippet]. Zen_ModLib is a shared library and needs no server install. |
| BuildingHealthDisplay 0.8.1 | Client [INFERRED]. | README says nothing about server or client. It is a UI-only fork of an aedenthorn mod. https://github.com/cjayride/BuildingHealthDisplay_Fork |
| GraphicsConfigPlus 1.3.1 | Client. | Graphics settings only. README has no server mention. |
| AAABuildMenu 1.0.4 | Client [INFERRED]. | UI-only. I found no server statement [snippet]. |
| BetterArchery 2.0.2 | Client, with a risk (see below). | The Nexus-derived note is that "Enable Bow Draw Movement Speed Reduction ... seemingly can't be enforced via the server" [snippet]. That implies local, per-player behaviour. |
| SatelliteMap 0.3.0 | Not "enforced" per its page. Server part is optional; clients are optional. | "Install it on the server and on every client that wants the picture. Clients without the mod just see the usual map; nothing is enforced. Without the mod on the server the picture still works, but shows only relief, water and shores: no buildings, paths or forest." "On a dedicated server the server's value applies to everyone." [snippet, https://thunderstore.io/c/valheim/p/Qua8ion/SatelliteMap/] |
| Server_devcommands 1.115.0 | Not strictly server-enforced per its page. | "Client side mod ... install on the admin client and optionally on the server." Some commands (event, find, randomevent, stopevent) need the server install. Usage is logged server-side only if both have it. [snippet, https://thunderstore.io/c/valheim/p/JereKuusela/Server_devcommands/] |

**Flags for the CORE/FULL split**
1. **SatelliteMap and devcommands are not "enforced" by their own pages.** Both say the server install is optional and clients without the mod are not rejected. Possible explanations are a Jotunn version check, or the owner intentionally putting them server-side. Either way, nothing in the docs says a CORE client would be kicked. Test with a CORE client and a FULL server.
2. **Two mods add custom items, which is the real misclassification risk.**
   - ATM adds a craftable Shovel. The README says it "will not affect existing shovels in the world" when disabled.
   - BetterArchery adds the Leather Quiver (crafted from 20 leather scraps, 10 deer hides and 10 troll hides, per the Thunderstore page via search).
   - "Client-only" means nobody is kicked. It does not mean the items are safe. A CORE player who opens a chest holding a shovel or quiver lacks the prefab. My expectation is that the item fails to deserialise and is silently lost on the next save **[INFERRED from general Valheim behaviour, not documented]**.
   - Mitigations: test it, tell FULL players not to store quivers or shovels in shared chests, or set `Shovel=false`, or move both mods to CORE. A generic hosting article also says item-adding mods are the ones that "kick people" [snippet, low.ms].
3. **The server-syncable mods are optional.** ATM, DiscoveryPins, QuickStackStore and Seasons settings can only be enforced if the mod is on the server. With no server install, each player's local config wins. For coherence, ship identical config files in the pack.
4. **Seasons is the strictest mod in the wider pack.** "Seasons must be installed on the server and every connecting client." Current season and day are server-controlled. https://github.com/shudnal/Seasons (raw README). Anything that changes visuals must tolerate Seasons on every client.

## 2. Overlap and redundancy

- **AAABuildMenu vs the vanilla 1.0 hammer menu (strong overlap).**
  - Vanilla 1.0 rebuilt the build menu "from scratch, with favorites and recent tabs plus a proper search function". You can search, or sort by recents, usage and material [snippet: valheimgame.com news, https://www.valheimgame.com/news/valheim-1-0-has-arrived-/].
  - AAABuildMenu on 1.0 adds Can Build, Most Used and Pinned tabs, a sort button, a resizable window and recipe pinning (Alt+click) with a material tracker. A setting restores the pre-1.0 menu [snippet, https://thunderstore.io/c/valheim/p/Azumatt/AAABuildMenu/].
  - Its "Pinned" overlaps vanilla Favorites and its "Most Used" overlaps vanilla sort-by-usage. It is the most redundant mod in the list if the goal is near-vanilla.
  - Keep it only for "Can Build" and the material tracker.
  - Search results disagree on maintenance: one snippet says the package is deprecated, another says actively maintained. Verify on the page.
- **DiscoveryPins and SatelliteMap, partial overlap.** SatelliteMap draws ruins and boss altars as terrain art. DiscoveryPins pins dungeons, ore, portals, and open-world locations (camps, villages) via hotkey. Double-marking is possible **[INFERRED]**.
- **ZenUI vs others.**
  - Its ammo-remaining display overlaps the BetterArchery quiver HUD (off by default).
  - Its coloured durability bars are separate from BuildingHealthDisplay (structure health).
  - Its crafting-panel replacement is not covered by the vanilla 1.0 hammer and serving-tray menu overhaul **[INFERRED]**.
  - Its feature list: visual crafting panel with grouping, sorting and search, coloured durability and food bars, ammo display, hide status effect icons, biome notifications, disable slide animation, auto-open skills, disable shout capitalisation [snippet]. Most of these are off-vanilla by design.
- **BuildingHealthDisplay.** Vanilla already shows a piece health bar on hover **[INFERRED from game knowledge]**, so this is recolour plus optional text plus structural-integrity readout, not a new system.
- **WhichModAddedThis.** It adds the mod name to item tooltips and the hammer HUD. That works against the "feels like vanilla" goal. Keep it as an admin or debug mod, or leave it in FULL only.
- **No real duplicates.** The following are not duplicated by anything else in the list: Gizmo, ComfyAutoRepair, QuickStackStore, MassFarming, UnRemove, ATM, GraphicsConfigPlus and Server_devcommands.
- **Beyond-vanilla additions to decide on.**
  - MassFarming: radius harvest, grid planting (5x5 default) and world pickups (rocks, sticks, berries, beehives).
  - ATM: shovel, square tools, precision raise, reset tool.
  - QuickStackStore: sort, trash, area restock.
  - BetterArchery: quiver, retrievable arrows, zoom.
  - Gizmo: free-axis rotation.
  - UnRemove: undo.

## 3. Settings for coherence

### ATM and the crater mod
Documented ATM facts (raw README, https://github.com/searica/TerrainTools):
- "All terrain operations, including resetting terrain modifications, work in multiplayer and are synced to other players."
- Adds a Shovel that lowers terrain, a "remove terrain modifications tool that lets you reset terrain", a precision raise tool, and square tools.
- `MaxRadius` default 10, range 4 to 20. `RadiusModKey` default LeftAlt, `SharpnessModKey` default LeftControl. Hover height info is on by default.
- Known issue: resetting at a zone edge with a big height difference can make the terrain "tear".
- The README's configuration tables use templated placeholder names such as `HoeToolName`, so check the generated `.cfg` for the real key names.

Suggestions (**[INFERRED]** unless stated):
- The reset tool and raise-ground can repair creature craters if the server mod stores them as standard terrain modifications. That is likely but unverified. It makes craters a nuisance rather than a lasting consequence.
- For a near-vanilla feel, consider `Shovel=false` (documented: stops crafting new shovels only), and possibly disabling the reset and precision tools.
- This is per-client, so ship the config in the pack.
- `MaxRadius` cannot be enforced without installing ATM on the server. Everything else is fine at default.
- **Key conflict.** Gizmo's default modifiers are LeftShift for X rotation and LeftAlt for Z rotation. ATM's radius modifier is LeftAlt with scroll wheel. Gizmo's README offers `[Ignored] ignoreTerrainOpPrefab` ("If enabled, rotation will be ignored for terrain-modifying prefabs"). Check that it is enabled, or remap `RadiusModKey`.
- Test on a cratered, hoe-levelled area: two systems write terrain deltas and the last writer wins **[INFERRED]**. This is also a possible perf concern if craters accumulate per zone.

### DiscoveryPins and SatelliteMap
- SatelliteMap: "Icons, marks, pings and map controls are untouched, with the picture lying under them" [snippet]. So there is no functional clash.
- `DetailLevel` runs 0 to 3: 0 is relief and water, 1 adds paths, fields, buildings, ruins and boss altars, 2 adds forest and large rocks, 3 shows everything down to bushes. **The server's value applies to everyone.** Level 1 is the most vanilla-adjacent. Levels 2 and 3 reveal forest and bushes and stray from vanilla. There is also an `AddDetailLimit` API for other mods [snippet].
- SatelliteMap only draws what the player's map already knows ("the picture never shows what your map does not know yet"), so it should not spoil exploration.
- For DiscoveryPins, consider turning off the open-world location auto-pin (camps, villages) so SatelliteMap and pins are not both marking the same things. Keep dungeon, ore, portal and death-pin cleanup if you want them.
- **Seasons vs SatelliteMap, the biggest unresolved risk.**
  - Seasons README: "minimap will be recolored using the seasonal colors setting"; `Custom biome settings.json` holds "winter colors for minimap" and needs a world restart.
  - Seasons also freezes water and spawns ice floes.
  - SatelliteMap redraws the map and minimap from world data. It mentions "how the ground was painted", but whether it reads Seasons' recolours, or frozen water, is undocumented.
  - **[INFERRED]** Seasons' minimap recolour and ice may be invisible under SatelliteMap. The map could then look summery in winter. Test this before finalising.

### GraphicsConfigPlus vs Seasons
- GCP default: "The default configuration uses the game's standard values." It "will override shadows, vSync, and distance/LOD options. Use the config file for those instead of the game menu."
- GCP Flavor and Post-processing options: bloom, lens dirt, sun shafts, gamma, SSAO intensity, chromatic aberration, fog, AA, disable-all post-processing.
- Seasons customises: fog density and luminance per time of day, ambient occlusion colour and intensity, aurora, `screenColorTemperature` (a global colour-grade shift), summer haze and heat distortion, plus a grass patch-size and density curve.
- **[INFERRED]** Flavor settings will stack with or fight Seasons' lighting. Limit GCP to performance settings and leave gamma, fog, bloom, AO and SSAO at default. Do not use "disable post-processing", which would remove some of the seasonal look.
- Grass density is Seasons' domain. Do not also tweak grass or vegetation LOD in GCP, or at least do not duplicate. Seasons' README warns about RenderLimits and FPS.
- Share one GCP config across the pack so shadow and LOD distance look the same for everyone.

### ZenUI scope
- Keep the minimum: coloured durability bars and possibly biome notifications.
- Disable what goes beyond vanilla: the crafting panel replacement (needs a logout to change), hide status effect icons, shout capitalisation and slide animation.
- **Caution:** hide-status-icons may hide Seasons' seasonal buff icon **[INFERRED]**. Seasons has its own setting to hide its buff and show day or time-left format.

### Smaller points
- **MassFarming vs Gizmo.** Both use LeftShift: MassFarming's default hotkey is Left Shift for mass harvest and grid planting, and Gizmo uses LeftShift for X rotation while placing. Gizmo lists saplings in its ignore-list example (`Beech_Sapling=1,...`), which suggests plants are affected **[INFERRED]**. Change one binding, or add your crop prefabs to `ignorePrefabNameList`. The README also mentions an "ignore stamina when mass planting" option. I have not verified its key name.
- **BetterArchery.** Beyond your known fix, `Set Arrow Velocity` (70 vs vanilla 60), `Set Arrow Gravity` (15 vs vanilla 5), aim direction and retrieve chances are per-client (v1.7.6 list). Ship identical values. Its quiver hotkeys (Alt+1/2/3) share Alt with Gizmo (Z axis), ATM (radius), AAABuildMenu (pin) and QuickStackStore (favourite, Alt by default).
- **QuickStackStore.** In multiplayer, area stacking and restocking are off by default: "disabled by default, but you can simply enable the config setting `AllowAreaStackingInMultiplayerWithoutMUC`". Decide on one value for everyone. Installing it on the server would sync it.

## 4. Integration ideas

**Documented**
- SatelliteMap `DetailLevel` is server-controlled, so the owner sets one look for everyone. Seasons settings marked `[Synced with Server]` sync from the server, with override via `ConditionalConfigSync.SyncPolicy.cfg` (README).
- Gizmo `ignoreTerrainOpPrefab` exists for exactly the ATM terrain-tool case.
- DiscoveryPins's synced settings can be locked from the server if it is installed there.
- Seasons has a global-key option for season state "disabled by default", for other mods to read.

**Inferred**
- Turn a player-visible toggle such as the shovel into a single pack config and treat crater repair as deliberate gameplay.
- Test order: one client in winter, SatelliteMap at level 1, on a cratered hillside.
- Show the crater mod's craters to SatelliteMap at 0.5 to 2 m per pixel so players can see damage on the map.
- Keep WhichModAddedThis out of player packs to preserve the vanilla illusion.

## 5. Known conflicts and load-order notes

- **BetterArchery:** known conflicts with ValheimPlus and Extended Player Inventory (disable the quiver) [snippet]. "Enable Quiver ... Don't change this value while in the game." Also the Balrond issue you already solved.
- **ATM:** may conflict with any other terrain-radius mod. Partial issues with ValheimPlus and FastTools (use ToolTweaks). The README says it is "fully compatible with EpicLoot".
- **Seasons:** incompatible with Seasonality by RustyMods. Needs ConditionalConfigSync. Changes to terrain colours require a season change to take effect.
- **Gizmo:** "Disable or uninstall any installed ComfyGizmo_v1.8.0 or earlier." Do not install Gizmo Reloaded or other Gizmo forks alongside it **[INFERRED]**.
- **Dependencies:** ATM, WhichModAddedThis and SatelliteMap require Jotunn. ZenUI needs Zen_ModLib. Seasons needs ConditionalConfigSync.
- **Load order:** nothing documented. BepInEx resolves dependencies itself.

## What still needs a direct look
I could not see the Thunderstore pages for AAABuildMenu, UnRemove, ZenUI (1.8.11 configs), BetterArchery 2.0.2 and SatelliteMap 0.3.0. Check their full READMEs for any "server", "both" or Jotunn-enforcement wording before finalising the split.

## Sources
- https://github.com/searica/TerrainTools
- https://github.com/searica/DiscoveryPins
- https://github.com/BruceOfTheBow/BruceComfyMods/tree/main/ComfyGizmo
- https://github.com/redseiko/ComfyMods/tree/main/ComfyAutoRepair
- https://github.com/Goldenrevolver/QuickStackStore
- https://github.com/Xeio/MassFarming
- https://github.com/MSchmoecker/WhichModAddedThis
- https://github.com/cjayride/BuildingHealthDisplay_Fork
- https://github.com/cjayride/GraphicsConfigPlus_Fork
- https://github.com/shudnal/Seasons
- https://thunderstore.io/c/valheim/p/Azumatt/AAABuildMenu/
- https://thunderstore.io/c/valheim/p/Qua8ion/SatelliteMap/
- https://thunderstore.io/c/valheim/p/ZenDragon/ZenUI/
- https://thunderstore.io/c/valheim/p/JereKuusela/Server_devcommands/
- https://thunderstore.io/c/valheim/p/ishid4/BetterArchery/
- https://thunderstore.io/c/valheim/p/Neobotics/UnRemove/
- https://www.valheimgame.com/news/valheim-1-0-has-arrived-/
