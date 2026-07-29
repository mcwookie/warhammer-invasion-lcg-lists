# Warhammer Invasion LCG Lists

This repo contains a full card list for the Warhammer Invasion Living Card Game by [Fantasy Flight Games](https://www.fantasyflightgames.com). A project I was working on required me to ingest the full card data, but I had trouble finding a definitive list of cards with details in an easy-to-use format.

This is a **data-only** repo — card text and stats, no card images/scans.

## Contents

* `whi_full.json` / `whi_full.csv` — same data, two formats, one row/object per card. **1,133/1,133 cards** have full detail.

  | Column | Description |
  |---|---|
  | `name` | Card name |
  | `edition` | Expansion/pack name |
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

## Sources

Source data came from the following sources. Many thanks to the creators:

* [Steam Warhammer Invasion HD](https://steamcommunity.com/sharedfiles/filedetails/?id=3663636921) Workshop mod by [Almighty Brushwagg](https://steamcommunity.com/profiles/76561198031041236).
* Card data from [deckbox.org](https://deckbox.org/games/whi/cards).
* CardGameDB (originally at <https://cardgamedb.com>, now redirecting to [Fantasy Flight Games](https://fantasyflightgames.com)). Archived data was accessed via the [Wayback Machine](https://web.archive.org/).

## License and data ownership

The **compilation** in this repo — the merging, cross-referencing, and formatting of card data pulled from the sources above into `whi_full.json` and `whi_full.csv` — is my own work. I release it under [CC0 1.0](LICENSE) (public domain dedication): use, adapt, or redistribute it for any purpose, with no attribution required.

That said, the **underlying card content** itself (card names, rules text, stats, flavor text, and the Warhammer Invasion LCG setting and characters) is **not** my work and not my intellectual property. It is © Games Workshop Limited and/or Fantasy Flight Publishing, Inc. (Fantasy Flight Games), all rights reserved. It's reproduced here for reference, archival, and fan-preservation purposes only, in the same spirit as other fan-run card databases for Fantasy Flight LCGs (e.g. MarvelCDB, ArkhamDB, RingsDB) — not as a claim of ownership over the game or its content.

This project is not affiliated with, sponsored by, or endorsed by Games Workshop or Fantasy Flight Games. If either rights holder has a concern about this repo, please open an issue or reach out and I'll address it, including taking the repo down if requested.
