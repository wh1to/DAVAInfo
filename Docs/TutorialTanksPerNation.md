# WOTB/TB — tutorialTanksPerNation.yaml

Reference for `tutorialTanksPerNation.yaml`, the config that drives
the "choose your nation" step of the in-game tutorial in World of
Tanks Blitz / Tanks Blitz.

It lists one starter tank per nation — the light or medium tank the
new player is offered when picking a country — and the localisation
keys for the congratulation prompts shown after the choice.

The file is read by the hangar tutorial system, not by the battle
client.

---

## Purpose

When a new account runs the tutorial, the game shows a screen with
one nation per slot. For each nation the client needs:

- a tank name to display (and its localisation key),
- a vehicle type label (Light / Medium),
- a numeric index that fixes the order on screen,
- a prompt string to show after the choice is confirmed.

`tutorialTanksPerNation.yaml` provides all of that.

---

## Top-level structure

    nationTankMap:
        <nationKey>:
            name:   "<internal tank name>"
            locKey: "#<locFile>:<tankName>"
            type:   "Light" | "Medium"
            index:  <int>
    prompts:
        1:  "<prompt loc key>"
        2:  "<prompt loc key>"
        ...
        10: "<prompt loc key>"

Two top-level keys: `nationTankMap` and `prompts`.

---

## `nationTankMap`

A dict of nation key -> starter tank descriptor. The nation key is
the same lowercase string used everywhere else in the game
(`ussr`, `usa`, `germany`, `uk`, `japan`, `china`, `france`,
`european`).

### Fields

| Field | Type | Meaning |
|-------|------|---------|
| name | str | Internal vehicle name, matching the .sc2 base name |
| locKey | str | Localisation key of the form `#<file>:<key>` |
| type | str | `"Light"` or `"Medium"` |
| index | int | Order in the nation list, 0-based |

### Current entries

| Nation | name | type | index |
|--------|------|------|-------|
| ussr | T-26 | Light | 0 |
| usa | M2_lt | Light | 1 |
| germany | PzII | Light | 2 |
| uk | GB69_Cruiser_Mk_II | Light | 3 |
| japan | Ha_Go | Light | 4 |
| china | Ch07_Vickers_MkE_Type_BT26 | Light | 5 |
| france | RenaultR35 | Medium | 6 |
| european | F01_Vickers_MkF | Light | 7 |

All current starter tanks are Light except France, which is Medium.

### `name` and `locKey`

`name` is the internal vehicle id. It is the same string used in:

- the vehicle's `.sc2` file base name,
- the vehicle folder under `3d/Tanks/<Nation>/`,
- the `previewWith` field of camouflages and skins.

`locKey` is the localisation key for the display name. The format is:

    #<locFile>:<key>

| Part | Meaning |
|------|---------|
| `#` | Marks the start of a localisation reference |
| `<locFile>` | Which localisation file to look in |
| `:` | Separator |
| `<key>` | The key inside that file |

Examples:

| locKey | Loc file | Key |
|--------|----------|-----|
| `#ussr_vehicles:T-26` | ussr_vehicles | T-26 |
| `#gb_vehicles:GB69_Cruiser_Mk_II` | gb_vehicles | GB69_Cruiser_Mk_II |
| `#european_vehicles:F01_Vickers_MkF` | european_vehicles | F01_Vickers_MkF |

Note that the UK uses `gb_vehicles`, not `uk_vehicles`. This is a
legacy naming quirk that appears throughout the game's localisation
files.

### `type`

Vehicle class label shown next to the tank icon. Only two values
appear in this file:

| Value | Meaning |
|-------|---------|
| `Light` | Light tank |
| `Medium` | Medium tank |

This is a tutorial-only label, not the full vehicle class system
(which also has Heavy, TD and SPG).

### `index`

Deterministic ordering key. Smaller index = earlier in the list.
The current file uses 0..7, one per nation. The index is used to
lay the nations out on screen and to persist the player's choice.

---

## `prompts`

A dict of int -> localisation key. Each key is the prompt shown
after the player confirms a nation. The keys are the full path
inside the localisation file, not just a leaf:

    hangarTutorial/selectNation/Congratulation/Prompt1
    hangarTutorial/selectNation/Congratulation/Prompt2
    ...
    hangarTutorial/selectNation/Congratulation/Prompt10

| Field | Meaning |
|-------|---------|
| key (int) | Prompt number |
| value (str) | Full path inside the tutorial localisation file |

The client picks the prompt by number. The file currently defines
prompts 1 through 10, which is enough for any current or future
nation count up to 10.

The `PromptN` strings are not tied to specific nations — the same
pool of 10 is used regardless of which nation the player picks.

---

## Localisation key format

A localisation key always starts with `#` and is resolved at
runtime. Format:

    #<file>:<key>

| Part | Notes |
|------|-------|
| file | Without extension, e.g. `ussr_vehicles` |
| key | The leaf key inside the file |

Some keys include nested paths with `/`, e.g. the tutorial prompts:

    hangarTutorial/selectNation/Congratulation/Prompt1

In that case the file part is implicit (the tutorial localisation
file) and the whole string after the file prefix is the nested key.

---

## Nation keys

The eight nation keys used here match the rest of the game:

| Key | Nation |
|-----|--------|
| ussr | Soviet Union |
| usa | United States |
| germany | Germany |
| uk | United Kingdom |
| japan | Japan |
| china | China |
| france | France |
| european | Pan-European tech tree |

The nation key is lowercase and is the same string used as the
first part of `previewWith` in `camouflages.yaml` and as the folder
name under `3d/Tanks/` (with different capitalisation).

---

## Nation prefix mapping

The nation keys map to the nation prefixes used in vehicle names
and folders:

| Nation key | Vehicle prefix | Folder |
|------------|----------------|--------|
| ussr | (none) | USSR |
| usa | (none) | USA |
| germany | (none) | German |
| uk | GB | UK |
| japan | (none) | Japan |
| china | Ch | China |
| france | (none) | France |
| european | F, S, Cz, Pl, It | European |

The prefix appears in the internal `name`, e.g. `GB69_Cruiser_Mk_II`
(UK), `Ch07_Vickers_MkE_Type_BT26` (China), `F01_Vickers_MkF`
(European).

---

## Example entry, expanded

    "germany":
        name: "PzII"
        locKey: "#germany_vehicles:PzII"
        type: "Light"
        index: 2;

Meaning:

- The German tutorial starter tank is the Panzer II.
- Its display name comes from the `germany_vehicles` localisation
  file, key `PzII`.
- It is a Light tank.
- It appears third in the nation list (index 2).

The trailing `;` after the index is a YAML quirk — in YAML a
semicolon is just part of a plain scalar, not a statement
terminator. Parsers that read the value as an int will see `2;`
and must strip the semicolon.

---

## Notes

- The file is intentionally tiny — one tank per nation — so the
  tutorial can present a simple choice without loading the whole
  vehicle database.
- France is the only nation whose starter is a Medium tank.
- The `;` at the end of every `index` line is not valid YAML for an
  integer. A strict parser will reject it; a lenient one will keep
  it as the string `"2;"`. Programs reading this file must strip the
  trailing `;` before parsing the number.
- `locKey` values are resolved at runtime by the localisation
  system. The importer never reads this file.
- The `prompts` section is a pool, not a nation-specific mapping.
  Any nation choice picks a prompt from this pool.
- The tutorial localisation file is separate from the vehicle
  localisation files. Vehicle names live in `<nation>_vehicles`,
  prompts live under `hangarTutorial/`.
- UK uses the `gb_` prefix in both the internal name and the
  localisation file name, even though the nation key is `uk`.
- The file has no version field. If the schema changes, the client
  is expected to be updated at the same time.
- Adding a new nation requires: a new entry under `nationTankMap`
  with a unique `index`, and (if the nation is new) a new
  localisation file. The `prompts` pool does not need to change.
