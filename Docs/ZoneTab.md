# tzdb — zone.tab

Reference for `zone.tab`, the timezone-to-location table shipped
with the IANA Time Zone Database (tzdb). It maps a country code to
one representative timezone, plus the geographic coordinates of that
timezone's main city.

---

## Purpose

`zone.tab` tells a program:

- which country a timezone belongs to,
- where (roughly) that timezone is on the globe,
- which zone name to hand to the OS timezone functions.

It is a plain text, tab-separated, ASCII-only file. It contains no
DST rules — those live in the compiled tzdata binaries.

---

## File format

    # comment lines start with '#'
    <country-code>  <coordinates>  <TZ>  [<comments>]

Each row has:

| Column | Meaning |
|--------|---------|
| country-code | ISO 3166-1 alpha-2, exactly one per row |
| coordinates | ISO 6709 latitude + longitude |
| TZ | IANA timezone name (e.g. `Europe/Berlin`) |
| comments | Optional free-text, mostly English region hints |

Columns are separated by a single TAB. Comments after the TZ column
are also separated by a TAB.

---

## ISO 6709 coordinates

The coordinates column uses ISO 6709 compact form:

    ±DDMM±DDDMM
    ±DDMMSS±DDDMMSS

| Part | Meaning |
|------|---------|
| ±DDMM | Latitude: sign, degrees, minutes |
| ±DDDMM | Longitude: sign, degrees, minutes |
| ±DDMMSS | Latitude with seconds |
| ±DDDMMSS | Longitude with seconds |

`+` means north / east, `-` means south / west. Latitude is always
2 digits of degrees, longitude is always 3. Seconds, when present,
are 2 digits.

Examples:

| Coordinates | Location |
|-------------|----------|
| +4230+00131 | 42°30'N 001°31'E — Andorra |
| -3436-05827 | 34°36'S 058°27'W — Buenos Aires |
| +513030-0000731 | 51°30'30"N 000°07'31"W — London |
| -7750+16636 | 77°50'S 166°36'E — McMurdo |

---

## Differences from zone1970.tab

| Feature | zone.tab | zone1970.tab |
|---------|----------|--------------|
| ASCII only | yes | no (uses UTF-8 comments) |
| One country per row | yes | a row can list multiple countries |
| Coverage since | 1970 | 1970 |
| Third column source | Zone or Link | Zone only |
| Multi-zone countries | one row per zone | one row per shared-rule group |

Because `zone.tab` has exactly one country per row, each row is the
intersection of a country and a timezone that has agreed with civil
time since 1970. Countries with several distinct timezones get
several rows.

---

## Country codes with multiple rows

| Country | Rows | Reason |
|---------|------|--------|
| AQ | 10 | Antarctic research stations |
| AR | 12 | Argentina's provinces |
| AU | 13 | Australian states and islands |
| BR | 15 | Brazilian states |
| CA | 28 | Canadian provinces and territories |
| CL | 4 | Chilean mainland, Aysén, Magallanes, Easter Island |
| CN | 2 | Beijing Time, Xinjiang Time |
| DE | 2 | Most of Germany, Büsingen |
| EC | 2 | Mainland, Galápagos |
| ES | 3 | Mainland, Ceuta/Melilla, Canary Islands |
| FM | 3 | Chuuk, Pohnpei, Kosrae |
| GL | 4 | Greenland regions |
| ID | 4 | Java/Sumatra, Borneo W/C, Borneo E/Sulawesi, Papua |
| KI | 3 | Gilbert, Phoenix, Line Islands |
| KZ | 7 | Kazakhstan's regions |
| MH | 2 | Most of Marshall Islands, Kwajalein |
| MX | 11 | Mexican states and border zones |
| MY | 2 | Peninsula, Sabah/Sarawak |
| NZ | 2 | Mainland, Chatham Islands |
| PF | 3 | Society, Marquesas, Gambier Islands |
| PG | 2 | Mainland, Bougainville |
| PS | 2 | Gaza Strip, West Bank |
| PT | 3 | Mainland, Madeira, Azores |
| RU | 26 | Russian federal subjects and MSK offsets |
| UA | 2 | Most of Ukraine, Crimea (see note) |
| UM | 2 | Midway, Wake |
| US | 29 | US states and territories |

---

## Special rows

### Antarctica

Antarctica is not a single timezone. Each research station uses a
zone tied to the country that operates it:

| Station | TZ | Operator |
|---------|----|----------|
| McMurdo | Antarctica/McMurdo | New Zealand |
| Casey | Antarctica/Casey | Australia |
| Davis | Antarctica/Davis | Australia |
| Dumont d'Urville | Antarctica/DumontDUrville | France |
| Mawson | Antarctica/Mawson | Australia |
| Palmer | Antarctica/Palmer | Chile |
| Rothera | Antarctica/Rothera | UK |
| Syowa | Antarctica/Syowa | Japan |
| Troll | Antarctica/Troll | Norway |
| Vostok | Antarctica/Vostok | Russia |

### Russia

Russia uses 26 rows, one per MSK offset band. The comment column
records the offset and the region, e.g.:

    RU  +554521+0373704  Europe/Moscow       MSK+00 - Moscow area
    RU  +5651+06036      Asia/Yekaterinburg  MSK+02 - Urals
    RU  +5301+15839      Asia/Kamchatka      MSK+09 - Kamchatka

The `MSK+NN` label is the offset from Moscow time, not from UTC.

### Ukraine / Crimea

The file contains a comment explaining a known limitation:

    # The obsolescent zone.tab format cannot represent
    # Europe/Simferopol well. Put it in RU section and list as UA.

    UA  +4457+03406  Europe/Simferopol  Crimea

`Europe/Simferopol` is listed under country code `UA` even though
it is physically in the RU section. The tzdb explicitly does not
take a position on territorial claims.

### China

China has two rows:

    CN  +3114+12128  Asia/Shanghai  Beijing Time
    CN  +4348+08735  Asia/Urumqi    Xinjiang Time

`Asia/Urumqi` is not an official Chinese timezone; it is kept for
historical and practical reasons.

---

## Zone name conventions

IANA zone names follow `<Area>/<Location>`:

| Area | Covers |
|------|--------|
| Africa | African countries |
| America | North and South America, Caribbean |
| Antarctica | Antarctic stations |
| Arctic | Svalbard |
| Asia | Asian countries |
| Atlantic | Atlantic islands |
| Australia | Australian states |
| Europe | European countries |
| Indian | Indian Ocean islands |
| Pacific | Pacific islands and territories |

Special names:

| Name | Meaning |
|------|---------|
| UTC | Coordinated Universal Time |
| Etc/GMT+N | Fixed offset, POSIX sign convention (inverted) |
| Etc/GMT-N | Fixed offset, POSIX sign convention (inverted) |

There is no `Asia/Beijing`; China's main zone is `Asia/Shanghai`.
There is no `Europe/Kiev` in newer tzdb; it is `Europe/Kyiv`.

---

## Example rows

    AD  +4230+00131  Europe/Andorra
    AE  +2518+05518  Asia/Dubai
    AF  +3431+06912  Asia/Kabul
    GB  +513030-0000731  Europe/London
    JP  +353916+1394441  Asia/Tokyo
    US  +404251-0740023  America/New_York  Eastern (most areas)

---

## How programs should use this file

1. Read rows, ignore lines starting with `#`.
2. Split on TAB. The first three fields are mandatory.
3. The fourth field, if present, is a human-readable hint.
4. Use the TZ field as the argument to `setenv("TZ", ...)`,
   `localtime`, `zoneinfo`, or the platform's equivalent.
5. Use the coordinates to place a marker on a map or to pick the
   closest zone to a user's GPS location.

Do not parse the comments field programmatically — it is free text
and its wording changes between tzdb releases.

---

## Notes

- The file is public domain.
- Entries whose third column is a Link (not a Zone) are still valid
  for setting the TZ environment variable.
- Coordinates are the location of the zone's representative city,
  not the centroid of the zone.
- The file does not contain DST rules, leap seconds or historical
  transitions. Those are in the compiled tzdata binaries.
- `zone1970.tab` is the preferred replacement. It differs mainly by
  allowing multiple country codes per row and by using UTF-8
  comments.
