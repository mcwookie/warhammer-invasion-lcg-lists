# Warhammer Invasion LCG Lists

This repo contains a full card list for the Warhammer Invasion Living Card Game by [Fantasy Flight Games](https://www.fantasyflightgames.com). A project I was working on required me to ingest the full card data, but I had trouble finding a definitive list of cards with details in an easy-to-use format.

This is a **data-only** repo — card text and stats, no card images/scans.

## Contents

* `whi_full.json` / `whi_full.csv` — same data, two formats, one row/object per card. **1,133/1,133 cards** have full detail.

  | Column | Description |
  |---|---|
  | `unique_id` | A stable 5-digit ID unique across every card in the game — see below |
  | `name` | Card name |
  | `pack_code` | A slug identifying the edition/expansion (e.g. `core-set`) — join against `whi_packs.json`'s `code` column for the full pack name and metadata |
  | `card_number` | Printed card number within its edition |
  | `race` | Faction/race (Dwarf, Empire, Orc, Chaos, High Elf, Dark Elf, Neutral) |
  | `type` | Card type (Unit, Support, Tactic, Quest, Legend, etc.) |
  | `cost` | Resource cost |
  | `power` / `health` | Unit combat stats, where applicable |
  | `loyalty` | Loyalty cost, where applicable |
  | `power_icons` | Power icon count, where applicable |
  | `traits` | Card traits/keywords |
  | `rules_text` | Full rules text |
  | `illustrator` | Credited artist, where known |
  | `card_quantity` | Print run quantity (copies per edition), where known |

  Not every column applies to every card type (e.g. a Tactic card has no `power`/`health`) — those cells are simply blank.

* `whi_packs.json` / `whi_packs.csv` — one row per edition/expansion (42 total).

  | Column | Description |
  |---|---|
  | `code` | Matches `pack_code` in `whi_full.json` |
  | `name` | Full pack name |
  | `position` | 1-based release order (Core Set is 1, then each expansion in the order it actually released) |
  | `total_unique` | Distinct card count in that edition — always populated |
  | `total_cards` | Physical print-run count (sum of every card's `card_quantity`) — blank for 6 editions where that data isn't fully known; see [Known limitations](#known-limitations) |
  | `available` | Release date, `YYYY-MM-DD` — see [The `available` field](#the-available-field) for how precise this really is |

### The `unique_id` field

Unlike MarvelCDB/ArkhamDB/RingsDB, this game has no official global card ID
— FFG's own printed `card_number` only resets to 1 at the start of each
**6-pack battle-pack cycle**, not per individual pack (e.g. Bleeding Sun,
the 6th pack of the Enemy cycle, is numbered 101-120 on the actual cards).

`unique_id` is a 5-digit `<2-digit set><3-digit card_number>` ID that
treats each 6-pack cycle as one "set" (matching how FFG already numbered
it), so `card_number` can be reused as-is with zero exceptions — every
cycle's `card_number` values already form a clean, non-overlapping 1-120
sequence across its 6 packs. Sets are ordered by real-world release date:

| Set | Contents |
|---|---|
| 01 | Core Set |
| 02 | The Corruption Cycle (6 packs) |
| 03 | Assault on Ulthuan |
| 04 | The Enemy Cycle (6 packs) |
| 05 | March of the Damned |
| 06 | The Morrslieb Cycle (6 packs) |
| 07 | Legends |
| 08 | The Capital Cycle (6 packs) |
| 09 | The Bloodquest Cycle (6 packs) |
| 10 | The Eternal War Cycle (6 packs) |
| 11 | Cataclysm |
| 12 | Hidden Kingdoms |

E.g. `01017` = Core Set card #17 (A Glorious Death); `04101` = Enemy cycle
card #101 (Oathbearer, printed on the Bleeding Sun pack).

Note this "set" grouping (12 groups, one per cycle) is different from
`whi_packs.json`'s `position` field (42 individual packs, true
chronological order) — a pack's `unique_id` set number and its release
`position` are two different numbering schemes for two different
purposes, not the same thing spelled two ways.

### The `available` field

Release order (`position`) and month came from Wikipedia's Warhammer:
Invasion article — BoardGameGeek, the usual source for exact release
dates, blocks automated access. **Exact release day is only confirmed
for 2 of the 42 packs** (Cataclysm: 2013-07-09, Hidden Kingdoms:
2013-11-26, both from Fantasy Flight's own "now available" announcement
pages). For every other pack, `available` uses the 1st of the release
month as a documented placeholder — it is **not** a claim that the pack
actually released on that literal day. Where a deluxe expansion released
in the same month as a battle pack (e.g. Assault on Ulthuan and The
Deathmaster's Dance, both March 2010), the ordering between them is a
judgment call, not a sourced fact.

## Sources

Source data came from the following sources. Many thanks to the creators:

* [Steam Warhammer Invasion HD](https://steamcommunity.com/sharedfiles/filedetails/?id=3663636921) Workshop mod by [Almighty Brushwagg](https://steamcommunity.com/profiles/76561198031041236).
* Card data from [deckbox.org](https://deckbox.org/games/whi/cards).
* CardGameDB (originally at <https://cardgamedb.com>, now redirecting to [Fantasy Flight Games](https://fantasyflightgames.com)). Archived data was accessed via the [Wayback Machine](https://web.archive.org/).

## Known limitations

* `whi_packs.json`'s `total_cards` is blank for 6 of the 42 editions — Core Set, The Skavenblight Threat, Assault on Ulthuan, The Burning of Derricksburg, Cataclysm, and Hidden Kingdoms — because `card_quantity` isn't populated for every card in those editions, and a partial sum would look like a real total without being one. `total_unique` has no such gap.
* CardGameDB/Wayback Machine coverage alone reaches 995/1,133 cards (88%); the Tabletop Simulator mod above filled the rest, bringing full detail to 1,133/1,133 (100%).

## License and data ownership

The **compilation** in this repo — the merging, cross-referencing, and formatting of card data pulled from the sources above into `whi_full.json`, `whi_full.csv`, `whi_packs.json`, and `whi_packs.csv` — is my own work. I release it under [CC0 1.0](LICENSE) (public domain dedication): use, adapt, or redistribute it for any purpose, with no attribution required.

That said, the **underlying card content** itself (card names, rules text, stats, flavor text, and the Warhammer Invasion LCG setting and characters) is **not** my work and not my intellectual property. It is © Games Workshop Limited and/or Fantasy Flight Publishing, Inc. (Fantasy Flight Games), all rights reserved. It's reproduced here for reference, archival, and fan-preservation purposes only, in the same spirit as other fan-run card databases for Fantasy Flight LCGs (e.g. MarvelCDB, ArkhamDB, RingsDB) — not as a claim of ownership over the game or its content.

This project is not affiliated with, sponsored by, or endorsed by Games Workshop or Fantasy Flight Games. If either rights holder has a concern about this repo, please open an issue or reach out and I'll address it, including taking the repo down if requested.
