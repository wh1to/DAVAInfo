# TB — camouflages.yaml

Reference for `camouflages.yaml`, the camouflage and skin registry
used by Tanks Blitz. Each entry describes one
paintable camouflage, animated camo, event skin, or whole-vehicle
skin preset: its texture, its UV tiling, its per-nation and per-tank
overrides, and (for skins) the custom entity slots it replaces.

The file is a single top-level dict. Every key is the camouflage's
internal id; every value is that camo's config block.

---

## Purpose

The client reads this file to know:

- which texture to apply as a decal or full repaint,
- how to tile, rotate, offset and colour that texture,
- which nations and which specific tanks get different tiling,
- whether the camo is animated and how,
- for skins: which custom entity slots on which tank get replaced.

The importer does not read this file directly. It is documented here
so the texture paths referenced by `texture`, `colorMap`, `normalMap`
and `rmMap` can be matched against files on disk.

---

## Entry categories

There is no explicit `category` field. The category is implied by the
shape of the entry:

| Category | How to recognise it |
|----------|---------------------|
| Paintable camo | Has `texture`, `scale`, `specularity`, optional `colorMap` / `normalMap` / `rmMap` |
| Animated camo | Same as above plus an `animation:` block and an `anim_*` id |
| Skin preset | Has `iconBig`, `preset`, `previewWith`, `type: 1`, usually `customEntities` |
| Skin override | Has `override:` only, points one tank at another skin's `preset` |
| Set / attachment | Has `customEntities:` with a `SlotTypeN` key and `type: 2` |

---

## Field reference

### Common fields (paintable camo)

| Field | Type | Meaning |
|-------|------|---------|
| texture | str | Albedo / decal texture path (no extension) |
| colorMap | str | Optional colour map applied over the base texture |
| normalMap | str | Optional normal map |
| rmMap | str | Optional roughness / metallic map |
| scale | [float, float] | UV tiling (U, V) applied to the texture |
| specularity | float | Specular strength, usually 0.4 |
| alpha | float | Overall opacity (0..1); used by foliage-style camos |
| albedoFlatColor | [r, g, b, a] | Flat tint multiplied onto the albedo |
| albedoFlatColorTrack | [r, g, b] | Same but only for tracks |
| decalTileColor | [r, g, b, a] | Tint applied to the decal layer only |
| decalTileRotation | float | Rotation in degrees |
| decalTileOffset | [float, float] | UV offset |
| decalTileScale | [float, float] | Extra scale on top of `scale` |
| decalTileShear | [float, float] | UV shear |
| decalNormalBlend | float | Blend weight of the normal map (0..1) |
| overrideTankNormalMap | bool | Replace the tank's normal map entirely |
| overrideTankRmMap | bool | Replace the tank's RM map entirely |

### Per-part overrides

Many entries add a `general:`, `gun:`, `hull:`, `mask:` or `turret:`
block that overrides the top-level values for that specific part:

    general:
        decalTileRotation: 30.0
        decalTileScale: [1.0, 1.0]
    gun:
        decalTileRotation: -45.0
    hull:
        decalTileRotation: 90.0
    mask:
        decalTileRotation: 45.0
    turret:
        decalTileRotation: 45.0

Each part block accepts any of the common fields above.

### Per-nation and per-tank overrides

    override:
        <nation>:
            albedoFlatColor: [...]
            scale: [...]
            specularity: ...
            override:
                <TankName>:
                    scale: [...]
                    alpha: ...
                    specularity: ...
                    general: { ... }
                    gun: { ... }
                    hull: { ... }
                    mask: { ... }
                    turret: { ... }

`<nation>` is one of `germany`, `ussr`, `usa`, `france`, `uk`,
`china`, `japan`, `european`, `other`. Only the fields present are
overridden.

### Animation block

    animation:
        isAnimated: bool
        aniMask: bool
        aniAmplitude: float
        middle: float
        power: float
        smooth: float
        speed: float
        waveLength: float
        waveSpeed: float
        width: float

| Field | Meaning |
|-------|---------|
| isAnimated | Master switch; if absent the camo is static |
| aniMask | Apply the animation only to the mask channel |
| aniAmplitude | Wave amplitude |
| middle | Wave midpoint |
| power | Wave exponent |
| smooth | Smoothing factor |
| speed | Overall animation speed |
| waveLength | Wavelength of the wave |
| waveSpeed | Speed of the wave along its axis |
| width | Wave width |

### Spreading block

Used by "progressive" camos that reveal over time (rank, topup,
victory-day).

    spreading:
        progressPower: float
        spreadingAxis: int
        spreadingBorderColor: [r, g, b, a]
        spreadingBorderSize: float
        spreadingDuration: float
        spreadingMode: int
        spreadingNoiseSize: float
        spreadingStart: float

### Skin preset fields

| Field | Type | Meaning |
|-------|------|---------|
| iconBig | str | Path to the big UI icon |
| preset | str | Preset id this skin installs |
| previewWith | str | `<nation>:<TankName>` the skin is designed for |
| shortUserString | str | Localisation key for the short name |
| userString | str | Localisation key for the full name |
| type | int | 1 = skin, 2 = attachment set |
| customEntities | dict | Map of slot name -> replacement item |

### customEntities

    customEntities:
        <SlotName>:
            item: <ItemName>
            hangarOnly: true | false

| Field | Meaning |
|-------|---------|
| item | The replacement entity to swap in |
| hangarOnly | If true, the swap only applies in the hangar, not in battle |

Slot names follow the vehicle's own skeleton slots, e.g.
`Skin_02_hull`, `Skin_02_turret_01`, `skin_gun_01`,
`turret_01_set`, `SlotType1`, `SlotType2`, `hull_Lamp_0`.

---

## Naming conventions

| Pattern | Meaning |
|---------|---------|
| `<name>` | Classic paintable camo (e.g. `alpine_fog`, `white_sands`) |
| `anim_<name>` | Animated camo |
| `battlepass<NN>_<MM>` | Battle Pass season camo |
| `battlepass_<year>_<month>_<NN>` | Year-based Battle Pass camo |
| `anim_cbr<year>_<NN>` | Championship Battle Royale animated camo |
| `anim_ny<year>_<NN>` | New Year animated camo |
| `anim_hlwn<year>_<NN>` | Halloween animated camo |
| `anim_cbrvillage_event` | Special event animated camo |
| `skin_<Tank>` | Whole-vehicle skin preset |
| `skin<N>_<Tank>` | Nth alternative skin for a tank |
| `skin2_<tank>` | Second skin slot for a tank |
| `set_<name>` | Attachment set (type 2) |
| `atlas_<name>` | Atlas camo (per-part textures) |

---

## Nations

| Nation key | Used by |
|------------|---------|
| germany | German tech tree |
| ussr | Soviet tech tree |
| usa | American tech tree |
| france | French tech tree |
| uk | British tech tree |
| china | Chinese tech tree |
| japan | Japanese tech tree |
| european | Pan-European tech tree |
| other | Crossover vehicles/tech tree |

---

## Camouflage catalogue (selection)

| Id | Category | Notes |
|----|----------|-------|
| alpine_fog | Paintable | France winter, has per-tank overrides |
| ancient_castle | Paintable | UK desert, alpha 0.63 |
| android_cam | Paintable | Common, has france overrides |
| anim_amber_meteorite | Animated | Common, animated normal + RM |
| anim_april_loyalty_2024_01 | Animated | April loyalty event |
| anim_big_boss | Animated | Common |
| anim_birthday_2023 | Animated | Common |
| anim_birthday_2024 | Animated | Common, spreading block |
| anim_blitz_point_01..04 | Animated | Blitz Point series |
| anim_blitztrix | Animated | Common, per-part overrides |
| anim_cbr2023_01..03 | Animated | Championship Battle Royale 2023 |
| anim_cbr2024_01 | Animated | Championship Battle Royale 2024 |
| anim_cbr2025_01..05 | Animated | Championship Battle Royale 2025 |
| anim_cbrvillage_event | Animated | Event camo |
| anim_china_art23_city | Animated | China art event |
| anim_compendium | Animated | Common |
| anim_digital_camo | Animated | Common, different gun texture |
| anim_dotd_2022 | Animated | Day of the Dead 2022 |
| anim_ev_skill | Animated | Common |
| anim_event_map_26_6 / 26_10 | Animated | Event map camos |
| anim_gravity_splash_01 / 02 | Animated | Common |
| anim_hexagon_camo | Animated | Common |
| anim_hlwn22_01..03 | Animated | Halloween 2022 |
| anim_hlwn23_01 | Animated | Halloween 2023 |
| anim_hlwn_2025_01 | Animated | Halloween 2025 |
| anim_hw_2025_01 / 02 | Animated | Halloween 2025 |
| anim_indep2023 | Animated | Independence 2023 |
| anim_lunar_new_year2022 | Animated | Lunar New Year 2022 |
| anim_mascarade | Animated | Common |
| anim_mgr | Animated | Common |
| anim_midas_1 | Animated | Common |
| anim_moon_city | Animated | Common |
| anim_moon_festival_2022 | Animated | Moon Festival 2022 |
| anim_ny2022_camo | Animated | New Year 2022 |
| anim_ny23_lunar_01 | Animated | Lunar New Year 2023 |
| anim_ny25_01 / 02 | Animated | New Year 2025 |
| anim_oktoberfest | Animated | Oktoberfest |
| anim_party_birthday_2022 | Animated | Birthday 2022 |
| anim_patrick23_coins | Animated | St. Patrick 2023 |
| anim_pb_01..03 | Animated | Common |
| anim_phosphorus_camo | Animated | Common |
| anim_podarki_kd_2025_04 | Animated | Gift event 2025 |
| anim_prisma_trm | Animated | Common |
| anim_pro_01 / 02 | Animated | Common |
| anim_rank_01..06 | Animated | Ranked rewards |
| anim_rating_battles_2022 / 2023 | Animated | Rating battles |
| anim_rating_sprint_25_1 / 26_3 | Animated | Rating sprint |
| anim_realistic_camo | Animated | Base realistic camo |
| anim_red_carbon | Animated | Common |
| anim_reinforced | Animated | Common, spreading |
| anim_sd1_* | Animated | Style "SD1" variants |
| anim_server_quest_2025 | Animated | Server quest 2025 |
| anim_space_star | Animated | Common |
| anim_stained_glass | Animated | Common |
| anim_steel_machine | Animated | Common |
| anim_summer_rush | Animated | Common |
| anim_t_defender_1 / 2 | Animated | T-Defender series |
| anim_tankopolia_2026_may / october | Animated | Tankopolia 2026 |
| anim_tankopolia_july_01 / 02 | Animated | Tankopolia July |
| anim_topup_01 / 02 _2025 | Animated | Top-up 2025 |
| anim_topup_2025 | Animated | Top-up 2025 |
| anim_unsurpassed | Animated | Common |
| anim_victoryday_2025 | Animated | Victory Day 2025 |
| anim_vkfest_26_6_01 / 02 | Animated | VK Fest |
| anim_wb_calendar | Animated | Calendar event |
| anim_wlprgs | Animated | Wargaming progression |
| apple_cam | Paintable | Common |
| ardent_ally | Paintable | Common |
| atari | Paintable | Common, crossover |
| atgm_fire | Paintable | USA only |
| atlas-ms-1-epic | Paintable | USSR MS-1 epic |
| atlas_* | Paintable | Atlas camo series |
| back_to_school | Paintable | Common |
| bastille | Paintable | France |
| bats2019 | Paintable | Multi-nation, complex override |
| battle_pass_july2020_1..4 | Paintable | Battle Pass July 2020 |
| battlepass12..40_* | Paintable | Battle Pass seasons 12 to 40 |
| battlepass_2020spy_1..4 | Paintable | Battle Pass 2020 spy season |
| battlepass_2024_* | Paintable | 2024 monthly camos |
| battlepass_2025_* | Paintable | 2025 monthly camos |
| battlepass_2026_* | Paintable | 2026 monthly camos |
| battlepass_amusement_* | Paintable | Amusement Park event |
| battlepass_feb2020_01..04 | Paintable | February 2020 |
| battlepass_frozen_* | Paintable | Frozen event camos |
| battlepass_gladiator_* | Paintable | Gladiator event camos |
| battlepass_ny2021_01..04 | Paintable | New Year 2021 |
| battlepass_ny21_* | Paintable | New Year 2021 extras |
| battlepass_robo_* | Paintable | Robo event camos |
| battlepass_space_* | Paintable | Space event camos |
| battlepass_victory_* | Paintable | Victory event camos |
| battleship | Paintable | Common |
| beast_plains | Paintable | USA summer |
| believe_in_yourself | Paintable | Germany summer |
| birthday2021_anim | Animated | Birthday 2021 |
| birthday_2019 / 2024 / 2_0_2024 | Paintable | Birthday camos |
| black_friday_camo | Paintable | Black Friday |
| black_predator_cam | Paintable | Common |
| bloody_night_cam | Paintable | Multi-nation tint |
| burned_cam | Paintable | Common |
| camo_70s | Paintable | 70s camo |
| camo_attempts_2026 | Paintable | Community attempts 2026 |
| camo_tournament_2025 | Paintable | Tournament 2025 |
| cat | Paintable | Common |
| cedar_undergrowth | Paintable | Common summer |
| centennial_oak | Paintable | USSR summer |
| china_3years_01 / 02 | Paintable | China 3rd anniversary |
| china_ancient_bamboo | Paintable | China |
| china_art23_oil_paint | Paintable | China art 2023 |
| china_black_dragon | Paintable | China |
| china_butterfly | Paintable | China |
| china_carp | Paintable | China |
| china_childrens_day_01 / 02 | Paintable | China Children's Day |
| china_exclusive_01..03_cam | Paintable | China exclusives |
| china_golden_phoenix | Paintable | China |
| china_ice_kirin | Paintable | China winter |
| china_little_duck_1 / 2 | Paintable | China |
| china_ny21_* | Paintable | China New Year 2021 |
| china_ny23_lunarny | Paintable | China Lunar New Year 2023 |
| china_porcelain | Paintable | China |
| china_snow_pands | Paintable | China winter |
| china_water_dragon | Paintable | China |
| china_white_tiger | Paintable | China |
| china_wise_turtle | Paintable | China |
| china_yb_pattern | Paintable | China |
| chinese_mt | Paintable | China |
| christmas2017 | Paintable | Christmas 2017 |
| clan_9 / gecko / rain / target / wolf | Paintable | Clan camos |
| cosmonautics_2024_apr_01 | Paintable | Cosmonautics 2024 |
| crash | Paintable | Common |
| cute_plates | Paintable | Common |
| cy_pink_line_anicamo | Animated | Ruby / Tanks Blitz |
| cy_trigon_anicamo | Animated | Ruby / Tanks Blitz |
| dark_woods | Paintable | Common, per-part textures |
| desert_fox | Paintable | USA desert |
| desert_oasis | Paintable | USSR desert |
| desert_tan | Paintable | Common desert |
| digital_blue | Paintable | Common |
| digital_green | Paintable | Common |
| dirty_weather | Paintable | UK winter |
| dry_fallen_trees | Paintable | Common summer |
| dunkirk | Paintable | France |
| effective_concealment | Paintable | Common |
| emerald_maze | Paintable | Common |
| esport_asia / eu / event / ru / usa | Paintable | Esports camos |
| euro_ole_ole | Paintable | Euro 2020 |
| european_d_01 / 02 | Paintable | European desert |
| european_s_01 / 02 | Paintable | European summer |
| european_w_01 / 02 | Paintable | European winter |
| event_activity_hb_2025 | Paintable | Event 2025 |
| event_map_26_10 / 26_6 | Paintable | Event map camos |
| f_camo | Paintable | Common |
| fest17 / fest18 | Paintable | Fest camos |
| festive_salute | Paintable | Birthday |
| fir_tree_ny2020 | Paintable | New Year 2020 |
| flight_of_the_butterfly | Paintable | Japan desert |
| foggy_albion | Paintable | UK winter |
| force_unity | Paintable | France summer |
| forest_of_thousand_souls | Paintable | Japan summer |
| freedom_spirit | Paintable | France summer |
| fv201_a45_ik | Skin override | UK FV201 A45 IK |
| fwc18_england / france / germany / japan / russia | Paintable | Football World Cup 2018 |
| galaktikos | Paintable | Common |
| gamescom2017 | Paintable | Gamescom 2017 |
| ghost | Paintable | Common |
| gold_cheap | Paintable | Common |
| gold_fund_2026 | Paintable | Gold fund 2026 |
| gold_lighting | Paintable | Common |
| green_clover | Paintable | UK summer |
| green_lightning_cam | Paintable | Common |
| green_shards | Paintable | Common |
| green_tundra | Paintable | Common |
| gw_fish_cam / gw_waves_cam | Paintable | Golden Week |
| hardbrightgreen_cam | Paintable | Common |
| hardcarbon_cam | Paintable | Common |
| hardcubes_cam | Paintable | Common |
| hardearned | Paintable | Common |
| harden_by_time | Paintable | Common |
| hidden_tiger | Paintable | Japan desert |
| hlwn2020_* | Paintable | Halloween 2020 |
| hlwn_2024_01 / 02 | Paintable | Halloween 2024 |
| hlwn_2025_01 | Paintable | Halloween 2025 |
| hw_dragon | Paintable | Halloween |
| ice_cougar | Paintable | USA winter |
| ice_shatters | Paintable | Common |
| ice_storm | Paintable | Common winter |
| ice_walker | Paintable | Common |
| icy_calm | Paintable | Germany winter |
| immortal_legion | Paintable | France desert |
| indep2019_cam / indep2020_cam | Paintable | Independence camos |
| indep22_01..04 | Paintable | Independence 2022 |
| inferno | Paintable | Common |
| isu130_cam | Skin override | USSR ISU-130 |
| july4th2021 | Paintable | July 4th 2021 |
| k9 | Paintable | Common |
| kaleidoscope_cam | Paintable | Common |
| killdeveloper | Paintable | Common |
| kosmo_cam | Paintable | Common |
| lava | Paintable | Common |
| liberte | Skin override | France AMX M4 1949 |
| lighting | Paintable | Common |
| line_cam | Paintable | Common |
| loyalty | Paintable | Common |
| loyalty_event24 | Paintable | Loyalty event 2024 |
| m-5-y_skin | Skin preset | USA M-5-Y |
| madgames2021_anim | Animated | Mad Games 2021 |
| may_9_2024 | Paintable | Victory Day 2024 |
| mgr_01 / mgr_02 | Paintable | Common |
| ms-1-rare | Paintable | USSR MS-1 rare |
| neon_bd_2020_cam | Paintable | Neon birthday 2020 |
| neon_cam | Paintable | Common |
| normandy_cam | Paintable | Common |
| ny2021_comic / cookie / mitten | Paintable | New Year 2021 |
| ny2023_03 / 04 | Paintable | New Year 2023 |
| ny2024_01 / 02 / htrs | Paintable | New Year 2024 |
| ny_2019_digital_1 / ice_2 / ice_3 | Paintable | New Year 2019 |
| ny_2025_quest | Paintable | New Year 2025 |
| ny_topup_2025 | Paintable | New Year top-up 2025 |
| oldsignora | Paintable | Common |
| olympic | Paintable | Common |
| orangehall_18 | Paintable | Halloween 2018 |
| oxidized_metal | Paintable | Common |
| pink_cam | Paintable | Common |
| point_line_cam | Paintable | Common |
| polygons_cam | Paintable | Common |
| power_of_will | Paintable | Germany summer |
| premium_acc | Paintable | Common |
| pride_of_the_nation | Paintable | Germany summer |
| profy_blue / profy_orange | Paintable | Common |
| pumpkin2019 | Paintable | Halloween 2019 |
| quests-camo-1 / 2 | Paintable | Quest camos |
| rating_battles_2023 | Paintable | Rating 2023 |
| rating_battles_platinum_2024 | Paintable | Rating 2024 |
| rating_battles_v4 | Paintable | Rating v4 |
| rating_battles_v4_anim | Animated | Rating v4 animated |
| rating_brilliant_19 | Paintable | Rating 2019 |
| rating_fight_01 | Paintable | Rating fight |
| rating_sprint | Paintable | Rating sprint |
| rayting | Paintable | Rating (alt id) |
| realistic_desert | Paintable | Realistic camo series |
| realistic_summer | Paintable | Realistic camo series |
| realistic_winter | Paintable | Realistic camo series |
| regular_digital | Paintable | Regular camo |
| remover | Paintable | Common |
| remover_26_7 | Paintable | Common |
| rlauncher | Paintable | Common |
| rock | Paintable | Common, animated |
| royal_ribbon | Paintable | UK desert |
| rpuncher | Paintable | Common |
| sandy_snake | Paintable | USA desert |
| scarlette_oriflamme | Paintable | France desert |
| scorching_barricade | Paintable | Common |
| scottish_mountain | Paintable | UK summer |
| sd1_* | Paintable | Style "SD1" variants |
| season23_* | Paintable | Season 23 camos |
| season_17 | Paintable | Season 17 camo |
| seasonone | Paintable | Season 1 camo |
| sentinel_ac_iv_ik | Skin override | UK Sentinel AC IV |
| set_* | Attachment set | Many Slots, type 2 |
| simplebrown_cam | Paintable | Common |
| simplegreen_cam | Paintable | Common |
| simplehexa_cam | Paintable | Common |
| skin2_* | Skin preset | Second skin slot |
| skin3_* | Skin preset | Third skin slot |
| skin_* | Skin preset | Whole-vehicle skins |
| slush | Paintable | Common winter |
| snow_boar | Paintable | USA winter |
| snow_lily | Paintable | France winter |
| snowy_forest | Paintable | USSR winter |
| space_storm | Paintable | Common |
| splinters_* | Paintable | Splinter camos |
| stars_ny2020_cam | Paintable | New Year 2020 |
| steel_diligence | Paintable | Germany desert |
| steppe_taiga | Paintable | USSR summer |
| still_strong | Paintable | Common |
| stlk_2026 | Paintable | Common 2026 |
| superbowl* | Paintable | Super Bowl event camos |
| swamp_alligator | Paintable | USA summer |
| swamp_moss | Paintable | Common summer |
| sweater_ny2020_cam | Paintable | New Year 2020 |
| tank_fest | Paintable | Tank Fest |
| tankopolia_2026_may / october | Paintable | Tankopolia 2026 |
| tgs_17 | Paintable | TGS 2017 |
| the_january_ice | Paintable | USSR winter |
| the_silence_of_the_mountains | Paintable | Japan winter |
| the_stony_soil | Paintable | Common |
| the_top_of_the_mountain | Paintable | Common winter |
| the_undeniable_courage | Paintable | Germany desert |
| tiger_event | Paintable | Tiger event |
| time_lines_cam | Paintable | Common |
| tournament | Paintable | Tournament |
| twistergex_18 | Paintable | Halloween 2018 |
| uprising2021_anim | Animated | Uprising 2021 |
| urban | Paintable | Common |
| user_made | Paintable | Common |
| ussr_T54_45_skin | Skin override | USSR T-54-45 |
| valiantwarrior | Paintable | Clan Wars |
| velvet_star | Paintable | Common, animated |
| voices_of_cicadas | Paintable | Japan summer |
| web2019 | Paintable | Multi-nation tint |
| webhall_18 | Paintable | Halloween 2018 |
| white_cherry | Paintable | Japan winter |
| white_sands | Paintable | Common desert |
| wild_dunes | Paintable | Common desert |
| wild_savannah | Paintable | USSR desert |
| winner_eu / winner_na | Paintable | Winner event camos |
| winter_cam | Paintable | Common winter |


## Notes

- Texture paths are relative to the `data/` root and have no file
  extension. The real file on disk is one of `.png`, `.dds`, `.tga`,
  `.pvr` or `.dvpl`.
- `colorMap` and `texture` are different: `texture` is the base decal,
  `colorMap` is an optional overlay that only makes sense on the PBR
  pipeline.
- `scale` is `[U, V]`. A value of `[1.0, 1.0]` means "one tiling
  across the whole hull"; smaller values tile the texture more.
- `specularity` is almost always `0.4` for non-metal camos. Metal /
  carbon camos use different values.
- `alpha` on a camo is the overall transparency. When present and
  below 1.0 the material must be alpha-blended.
- Animated camos (`anim_*`, and a few named ones) always carry an
  `animation:` block. If `isAnimated` is absent the animation block is
  ignored.
- Skin presets (`type: 1`) are not paint jobs — they swap whole
  entity slots via `customEntities`. Paintable camos never have
  `customEntities`.
- Attachment sets (`type: 2`) attach a decorative item (flag, drone,
  lamp, weapon) to a slot. They do not change the tank's paint.
- `previewWith` uses the form `<nation>:<TankName>` and tells the
  hangar which tank to show the item on.
- Nation keys inside `override:` are lowercase and match the game's
  nation folder names, except `european` which is the pan-European
  tech tree.
- Some entries contain both a top-level `texture` and a full
  `override:` tree — the overrides take priority per-nation and
  per-tank.
- `decalTileRotation` is in degrees, positive is counter-clockwise.
- `hangarOnly: true` on a `customEntities` slot means the swap is
  cosmetic in the hangar only and does not appear in battle.
