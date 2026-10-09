# WOTB/TB — maps.yaml

Reference for `maps.yaml`, the map registry used by World of Tanks
Blitz / Tanks Blitz. Each entry describes one playable map: its
internal id, its SC2 scene file, its UI sprite, its game modes, its
camouflage kind, and the spawn point clusters used by Assault mode.

The file is a single top-level `maps:` dict. Every key is the map's
internal name; every value is that map's config block.

Maps taken from Ruby / Tanks Blitz (e.g. `mars`, `cybermap`,
`iceworld`, `holmeisk`, `glacier`) are included here the same way
as the older PC / Blitz maps — same schema, same fields.

---

## Purpose

The client reads this file to know:

- which .sc2 to load for a given map id,
- which camouflage set the vehicles should use,
- which game modes the map supports,
- where each team spawns in Assault mode,
- which supremacy (capture) threshold applies.

The importer does not read this file directly. It is documented here
so the .sc2 paths referenced by `localName` can be matched against
files on disk.

---

## Top-level structure

    maps:
        <mapKey>:
            id:                     <int>
            tags:                   "<str>"          (optional)
            localName:              "<relative .sc2 path>"
            avaliableInTrainingRoom: <bool>
            levels:                 [<int>, ...]     (optional)
            spriteFrame:            <int>
            supremacyPointsThreshold: <int>
            availableModes:         [<int>, ...]
            vehicleCamouflageKind:  "summer" | "winter" | "desert"
            shadowMapsAvailable:    <bool>
            assaultRespawnPoints:
                allies:
                -   respawnNumber: <int>
                    points:
                    - [x, y]
                    ...
                enemies:
                -   respawnNumber: <int>
                    points:
                    - [x, y]
                    ...

---

## Field reference

| Field | Type | Meaning |
|-------|------|---------|
| id | int | Numeric map id used by the game client |
| tags | str | Short internal tag (e.g. `dt1`, `er0`, `mr1`, `gl1`) |
| localName | str | Relative path to the map's .sc2 scene file |
| avaliableInTrainingRoom | bool | Whether the map is selectable in the Training Room (note the original spelling) |
| levels | list[int] | Optional tier filter — map only appears for these vehicle levels |
| spriteFrame | int | Index of the map's icon in the UI sprite atlas |
| supremacyPointsThreshold | int | Capture points needed to win Supremacy mode |
| availableModes | list[int] | 0 = Supremacy, 1 = Assault |
| vehicleCamouflageKind | str | `summer`, `winter` or `desert` — applied to vehicles |
| shadowMapsAvailable | bool | Whether the map uses shadow maps |
| assaultRespawnPoints | dict | Spawn clusters for Assault mode, split by team |

---

## `localName` path convention

Map scenes live under:

    <data_root>/3d/Maps/<mapFolder>/<mapFile>.sc2

Folder and file use a short code: e.g. `17_karelia_ka/17_karelia_ka.sc2`.
The `tags` field, when present, is the short code used by other
config files (e.g. `dt1` = Desert Train).

Some maps share one .sc2 between multiple entries — for example
`desert_train`, `desert_train_02` and `desert_train_03` all point at
`02_desert_train_dt/02_desert_train_dt.sc2`. The entries differ only
in spawn points, modes and tier limits.

---

## Game modes

| `availableModes` value | Mode |
|------------------------|------|
| 0 | Supremacy (capture points) |
| 1 | Assault (attack / defend) |

A map with `[0, 1]` supports both. A map with `[0]` supports only
Supremacy. Assault-only maps and event maps (e.g. `mars_br`,
`glacier_br`, `mars_02`, `mars_03`) exist for special modes.

---

## Camouflage kind

`vehicleCamouflageKind` selects which camo textures the vehicles use
on that map.

| Value | Used on |
|-------|---------|
| summer | Default green / brown maps |
| winter | Snow maps (medvedkovo, himmelsdorf, malinovka, canyon, faust, glacier) |
| desert | Sand maps (desert_train, mountain, savanna, mars) |

---

## Assault spawn points

`assaultRespawnPoints` is split into `allies` and `enemies`. Each side
has one or more respawn groups, each with a `respawnNumber` and a
list of `points`. A point is `[x, y]` in the map's local 2D space
(the same space as the terrain bbox).

Maps can have more than the usual two groups per side:
- 1 group per side — simple maps,
- 2 groups per side — most maps, giving attackers a fallback spawn,
- 3 groups per side — `neptune` only.

Spawn coordinates are not normalised; they match the map's own
world units and can be negative.

---

## Tier limits

`levels` (when present) lists the vehicle tiers the map is allowed
for. Example: `lagoon` has `levels: [5, 6, 7, 8, 9, 10]` — it only
appears for tiers 5 to 10. Maps without `levels` have no tier
restriction.

---

## Map catalogue

| Key | id | tags | localName | Modes | Camo |
|-----|----|------|-----------|-------|------|
| karelia | 1 | — | 17_karelia_ka/17_karelia_ka.sc2 | 0,1 | summer |
| desert_train | 2 | dt1 | 02_desert_train_dt/02_desert_train_dt.sc2 | 0,1 | desert |
| erlenberg | 3 | er0 | 03_erlenberg_er/03_erlenberg_er.sc2 | 0,1 | summer |
| karieri | 4 | — | 23_karieri_kr/23_karieri_kr.sc2 | 0,1 | summer |
| amigosville | 5 | am1 | 05_amigosville_am/05_amigosville_am.sc2 | 0,1 | summer |
| rudniki | 6 | rd0 | 06_rudniki_rd/06_rudniki_rd.sc2 | 0 | summer |
| medvedkovo | 7 | md1 | 04_medvedkovo_md/04_medvedkovo_md.sc2 | 0,1 | winter |
| fort | 8 | — | 07_fort_ft/07_fort_ft.sc2 | 0,1 | summer |
| himmelsdorf | 9 | hm1 | 19_himmelsdorf_hm/19_himmelsdorf_hm.sc2 | 0,1 | winter |
| mountain | 10 | — | 21_mountain_mnt/21_mountain_mnt.sc2 | 0,1 | desert |
| savanna | 11 | — | 09_savanna_sv/09_savanna_sv.sc2 | 0,1 | desert |
| plant | 12 | — | 11_plant_pn/11_plant_pn.sc2 | 0,1 | summer |
| idle | 13 | — | 08_idle_id/08_idle_id.sc2 | 0,1 | summer |
| holland | 14 | — | 16_holland_hl/16_holland_hl.sc2 | 0,1 | summer |
| port | 15 | — | 14_port_pt/14_port_pt.sc2 | 0,1 | summer |
| malinovka | 19 | — | 12_malinovka_ma/12_malinovka_ma.sc2 | 0,1 | winter |
| pliego | 20 | — | 13_pliego_pl/13_pliego_pl.sc2 | 0,1 | summer |
| canal | 21 | cn0 | 18_canal_cn/18_canal_cn.sc2 | 0,1 | summer |
| italy | 23 | — | 22_italy_it/22_italy_it.sc2 | 0,1 | summer |
| milbase | 25 | mlb1 | 24_milibase_mlb/24_milibase_mlb.sc2 | 0,1 | summer |
| lagoon | 26 | — | 15_lagoon_ln/15_lagoon_ln.sc2 | 0,1 | summer |
| canyon | 27 | — | 25_canyon_ca/25_canyon_ca.sc2 | 0,1 | winter |
| rock | 30 | — | 28_rock_rc/28_rock_rc.sc2 | 0,1 | summer |
| grossberg | 31 | — | 30_grossberg_sh/30_grossberg_sh.sc2 | 0,1 | summer |
| skit | 35 | — | 29_skit_sk/29_skit_sk.sc2 | 0,1 | summer |
| lumber | 38 | lm0 | 31_lumber_lm/31_lumber_lm.sc2 | 0,1 | summer |
| faust | 39 | — | 32_faust_fa_night/32_faust_fa_night.sc2 | 0,1 | winter |
| forgecity | 40 | — | 34_forgecity_fc/34_forgecity_fc.sc2 | 0,1 | summer |
| neptune | 42 | nt0 | 33_neptune_nt/33_neptune_nt.sc2 | 0,1 | summer |
| rift | 43 | — | 35_rift_rt/35_rift_rt.sc2 | 0,1 | summer |
| desert_train_02 | 44 | dt2 | 02_desert_train_dt/02_desert_train_dt.sc2 | 0 | — |
| desert_train_03 | 45 | dt3 | 02_desert_train_dt/02_desert_train_dt.sc2 | 0 | — |
| medvedkovo_02 | 46 | md2 | 04_medvedkovo_md/04_medvedkovo_md.sc2 | 0 | — |
| medvedkovo_03 | 47 | md3 | 04_medvedkovo_md/04_medvedkovo_md.sc2 | 0 | — |
| amigosville_02 | 48 | am2 | 05_amigosville_am/05_amigosville_am.sc2 | 0 | — |
| amigosville_03 | 49 | am3 | 05_amigosville_am/05_amigosville_am.sc2 | 0 | — |
| milbase_02 | 50 | mlb2 | 24_milibase_mlb/24_milibase_mlb.sc2 | 0 | — |
| milbase_03 | 51 | mlb3 | 24_milibase_mlb/24_milibase_mlb.sc2 | 0 | — |
| milbase_04 | 52 | mlb4 | 24_milibase_mlb/24_milibase_mlb.sc2 | 0 | — |
| milbase_05 | 53 | mlb5 | 24_milibase_mlb/24_milibase_mlb.sc2 | 0 | — |
| erlenberg_01 | 54 | er1 | 03_erlenberg_er/03_erlenberg_er.sc2 | 0 | — |
| erlenberg_02 | 55 | er2 | 03_erlenberg_er/03_erlenberg_er.sc2 | 0 | — |
| canal_01 | 56 | cn1 | 18_canal_cn/18_canal_cn.sc2 | 0 | — |
| lumber_01 | 57 | lm1 | 31_lumber_lm/31_lumber_lm.sc2 | 0 | — |
| neptune_01 | 58 | nt1 | 33_neptune_nt/33_neptune_nt.sc2 | 0 | — |
| neptune_02 | 59 | nt2 | 33_neptune_nt/33_neptune_nt.sc2 | 0 | — |
| rudniki_01 | 60 | rd1 | 06_rudniki_rd/06_rudniki_rd.sc2 | 0 | — |
| rudniki_02 | 61 | rd2 | 06_rudniki_rd/06_rudniki_rd.sc2 | 0 | — |
| rudniki_03 | 62 | rd3 | 06_rudniki_rd/06_rudniki_rd.sc2 | 0 | — |
| moon | 70 | — | 40_moon_mn/40_moon_mn.sc2 | 0 | — |
| holmeisk | 71 | — | 26_holmeisk_hk/26_holmeisk_hk.sc2 | 0,1 | — |
| iceworld | 72 | — | 41_iceworld_ic/41_iceworld_ic.sc2 | 0 | — |
| glacier | 73 | gl1 | 42_glacier_gl/42_glacier_gl.sc2 | 0,1 | winter |
| glacier_br | 74 | — | 42_glacier_gl/42_glacier_br.sc2 | 0,1 | — |
| mars | 75 | mr1 | 01_mars_mr/01_mars_mr.sc2 | 0,1 | desert |
| mars_br | 76 | — | 01_mars_mr/01_mars_br.sc2 | 0,1 | — |
| himmelsdorf_02 | 77 | hm2 | 19_himmelsdorf_hm/19_himmelsdorf_hm.sc2 | 0,1 | — |
| himmelsdorf_03 | 78 | hm3 | 19_himmelsdorf_hm/19_himmelsdorf_hm.sc2 | 0,1 | — |
| himmelsdorf_04 | 79 | hm4 | 19_himmelsdorf_hm/19_himmelsdorf_hm.sc2 | 0,1 | — |
| mars_02 | 80 | mr2 | 01_mars_mr/01_mars_mr.sc2 | 0 | — |
| mars_03 | 81 | mr3 | 01_mars_mr/01_mars_mr.sc2 | 0 | — |
| glacier_02 | 82 | gl2 | 42_glacier_gl/42_glacier_gl.sc2 | 0 | — |
| glacier_03 | 83 | gl3 | 42_glacier_gl/42_glacier_gl.sc2 | 0 | — |
| cybermap | 84 | — | 20_cybermap_cb/20_cybermap_cb.sc2 | 0,1 | summer |

Entries with `—` in the Camo column omit `vehicleCamouflageKind`;
those are event or alternate variants that inherit the base map's
setting.

---

## Ruby / Tanks Blitz maps

These maps come from the Ruby / Tanks Blitz side of the game and are
present in the same registry:

| Key | id | LocalName | Notes |
|-----|----|-----------|-------|
| mars | 75 | 01_mars_mr/01_mars_mr.sc2 | Desert camo, both modes |
| mars_br | 76 | 01_mars_mr/01_mars_br.sc2 | Separate battle-royale SC2 |
| mars_02 | 80 | 01_mars_mr/01_mars_mr.sc2 | Supremacy-only variant |
| mars_03 | 81 | 01_mars_mr/01_mars_mr.sc2 | Supremacy-only variant |
| cybermap | 84 | 20_cybermap_cb/20_cybermap_cb.sc2 | Summer camo, both modes |
| iceworld | 72 | 41_iceworld_ic/41_iceworld_ic.sc2 | Supremacy-only |
| holmeisk | 71 | 26_holmeisk_hk/26_holmeisk_hk.sc2 | Both modes |
| glacier | 73 | 42_glacier_gl/42_glacier_gl.sc2 | Winter camo, both modes |
| glacier_br | 74 | 42_glacier_gl/42_glacier_br.sc2 | Separate battle-royale SC2 |
| glacier_02 | 82 | 42_glacier_gl/42_glacier_gl.sc2 | Supremacy-only variant |
| glacier_03 | 83 | 42_glacier_gl/42_glacier_gl.sc2 | Supremacy-only variant |

`mars_br` and `glacier_br` are separate .sc2 files (`01_mars_br.sc2`,
`42_glacier_br.sc2`) used by the battle-royale mode; the regular
maps reuse the base SC2.

---

## Notes

- `avaliableInTrainingRoom` is spelled with a single `l` in the
  original file. Do not "fix" it — the client reads this exact key.
- `spriteFrame` values are not unique. Several maps reuse the same
  icon (e.g. all Desert Train variants use frame 2, all Milbase
  variants use frame 17, all Himmelsdorf variants use frame 9).
- Spawn coordinates are in the map's own local space and match the
  terrain bbox used by Landscape.py.
- The importer uses `localName` only as a hint when locating the
  .sc2 to load; it never reads maps.yaml itself.
- The `id` field is the value the client passes around internally;
  it is not the same as the number in `spriteFrame` or the folder
  prefix (e.g. `17_` in `17_karelia_ka`).
