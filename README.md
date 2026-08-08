# warhammer-invasion-cards

A small scraper that compiles a complete card list for **Warhammer: Invasion
LCG** (Fantasy Flight Games / Games Workshop, 2009–2013, discontinued) into
JSON and CSV. No official API or community database exists for this game
(unlike, say, MarvelCDB or ArkhamDB for other FFG LCGs), so this pulls
from the two remaining sources that still host the data.

## What it does

Three phases:

1. **Base listing** — scrapes [deckbox.org](https://deckbox.org/games/whi/cards)'s
   full card table: name, race, type, cost, and edition for every
   card/printing (1,133 rows across all sets).
2. **Full detail** — cardgamedb.com itself is now behind an active
   Cloudflare bot challenge and unreachable to a plain HTTP client (even the
   individual card permalinks that used to work). Instead, this phase
   queries the [Wayback Machine](https://web.archive.org)'s CDX API for
   archived (2012–2013) snapshots of CardGameDB's card detail pages, fetches
   each one, and parses card name/edition (from the page `<title>`) plus
   type, race, cost, loyalty, card number, quantity, illustrator, and rules
   text from the labeled fields in the body. Parsed records are matched back
   to the base listing by (name, edition), with a fuzzy-matching fallback
   for near-miss spellings/punctuation.
3. **Gap-fill from a Tabletop Simulator mod** (optional, `--tts-json`) —
   community TTS mods for the game store each card's full rules text
   directly on the card object's `Description` field, independent of both
   Deckbox and CardGameDB. Passing a TTS save/mod JSON parses that out and
   fills in whatever phase 2 couldn't — see
   [Known limitations](#known-limitations) for match-rate details.

## Usage

```bash
pip install requests beautifulsoup4
python3 scrape_whi.py                # full run, both phases
python3 scrape_whi.py --base-only    # deckbox.org pass only
python3 scrape_whi.py --skip-base    # reuse existing whi_base.csv, rerun detail pass
python3 scrape_whi.py --skip-wayback # reuse whi_wayback_cache.json instead of
                                      # re-fetching from web.archive.org (fast
                                      # re-match after tweaking EDITION_ALIASES
                                      # or match_key)
python3 scrape_whi.py --delay 1.5    # slower, gentler on web.archive.org
python3 scrape_whi.py --skip-base --skip-wayback \
    --tts-json /path/to/mod.json     # fill remaining gaps from a TTS save
```

## Output files

| File | Contents |
|---|---|
| `whi_base.csv` | name, race, type, cost, edition — one row per card/printing |
| `whi_wayback_cache.json` | every parsed CardGameDB record recovered from the Wayback Machine, before matching — lets you re-run the matching step without re-fetching |
| `whi_full.json` / `whi_full.csv` | base fields merged with rules text, card number, quantity, loyalty, illustrator (CardGameDB) and power/health/traits (TTS-only fields) where a match was found — same data, two formats (the CSV is flat, one row per card, with empty cells for whichever fields don't apply to a given card type). `pack_code` (a slug, e.g. `core-set`) identifies the edition — join against `whi_packs.json`'s `code` to get the full pack name/metadata |
| `whi_image_match.csv` | unique_id, name, pack_code, card_number only — the minimal fields needed to match against scanned card images |
| `whi_packs.json` / `whi_packs.csv` | one row per edition/expansion (42 total) — `code` (matches `pack_code` in `whi_full.json`/`whi_image_match.csv`, and the edition-slug in `rename_warhammer_invasion.py`'s output filenames), `name` (full pack name), `position` (1-based release order), `total_unique` (distinct card count), `total_cards` (physical print-run count — blank for 6 editions, see below), `available` (release date, `YYYY-MM-DD`) — see [The `available` field](#the-available-field) |
| `whi_detail_failures.csv` | base rows with no matching archived detail page |
| `whi_wayback_unmatched.csv` | archived detail pages that didn't match any base row — should normally be empty; includes both `wayback_url` (the working, clickable archived page) and `source_url` (the original live cardgamedb.com URL, kept only for citation — it now redirects to a Cloudflare-protected page and won't load) |

## The `unique_id` field

The game has no official global card ID scheme the way MarvelCDB/ArkhamDB/
RingsDB do for other FFG LCGs (their `code` field is unique across the
*entire* card pool). FFG's own printed `card_number` for Warhammer:
Invasion only resets to 1 at the start of each **6-pack battle-pack
cycle** — it does not reset per individual pack. So Bleeding Sun (the 6th
pack of the Enemy cycle) is numbered 101-120 on the actual cards, not
1-20.

`unique_id` turns this into a stable 5-digit ID, `<2-digit set><3-digit
card_number>`, by treating each 6-pack cycle as **one set** (matching how
FFG already numbered it) rather than six:

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

Sets are ordered by real-world release date. `card_number` is reused
as-is (zero-padded to 3 digits) for the setnumber — verified this works
with **zero exceptions**: every cycle's `card_number` values already form
a clean, non-overlapping 1-120 sequence across its 6 packs, so no
re-numbering is ever needed. E.g. `01017` = Core Set card #17 (A Glorious
Death); `04101` = Enemy cycle card #101 (Oathbearer, printed on Bleeding
Sun). See `EDITION_SET_NUMBER` / `compute_unique_id()` in `scrape_whi.py`.

## The `available` field

Release order (`position`) and month came from
[Wikipedia's Warhammer: Invasion article](https://en.wikipedia.org/wiki/Warhammer:_Invasion)
— BoardGameGeek, the usual source for exact release dates, blocks both
its website and its XML API from automated fetches. **Exact release day
is only confirmed for 2 of the 42 packs** (Cataclysm: 2013-07-09, Hidden
Kingdoms: 2013-11-26, both from Fantasy Flight's own "now available"
announcement URLs). For every other pack, `available` uses the 1st of the
release month as a documented placeholder — it is **not** a claim that
the pack actually released on that literal day. Where a deluxe expansion
released in the same month as a battle pack (e.g. Assault on Ulthuan and
The Deathmaster's Dance, both March 2010), the ordering between them is a
judgment call, not a sourced fact. See `PACK_RELEASE_ORDER` in
`scrape_whi.py` if you're able to source more precise dates.

## Known limitations

- `whi_packs.json`'s `total_cards` (physical print-run count, i.e. the sum
  of every card's `card_quantity` in that edition) is **blank for 6 of the
  42 editions** — Core Set, The Skavenblight Threat, Assault on Ulthuan,
  The Burning of Derricksburg, Cataclysm, and Hidden Kingdoms — because
  `card_quantity` is CardGameDB-only and isn't populated for every card in
  those editions. Rather than sum only the cards that happen to have it
  (which would silently produce an undercount that looks like a real
  total), `build_pack_records()` leaves it blank whenever coverage is
  incomplete. `total_unique` (distinct card count) has no such gap — it's
  always fully populated.
- CardGameDB/Wayback alone matches **995/1,133 (88%)** — the rest is a
  true archive gap, not a naming/parsing issue (some expansions have
  little or no Wayback coverage of their individual card pages; *Hidden
  Kingdoms* has none at all). Adding a TTS mod as a gap-fill source
  (`--tts-json`) closes the rest: **1,133/1,133 (100%)** have full detail.
  (One card, *Ariel, Queen in the Woods*, needed a manual fix first —
  Deckbox's own source data had it as "Ariel. Queen in the Woods", a
  period instead of a comma, which kept it from matching the TTS mod's
  correctly-spelled entry. Visually confirmed against a physical card and
  corrected directly in `whi_base.csv`.)
  - Naming mismatches between Deckbox and CardGameDB (a leading "The", a
    hyphen vs. a space, "Core" vs. "Core Set", spelling variants, stray
    punctuation, and the "Alliance" cards' different title layout) are
    already handled by `match_key()`'s normalization plus a fuzzy-matching
    fallback (`fuzzy_match_row()`) that only matches within the same
    edition and requires a clear best candidate. `whi_wayback_unmatched.csv`
    should come out empty; if a future run leaves rows in it, that's a new
    naming pattern worth adding to `EDITION_ALIASES` or the parser.
  - Card number is **not** required for image matching: every (name,
    edition) pair in `whi_base.csv` is already unique (verified — zero
    duplicates), so `whi_image_match.csv` was fully usable even before the
    TTS gap-fill.
  - A specific TTS mod is not bundled with this repo (it's a large,
    separately-sourced save file) — `--tts-json` is opt-in and the exact
    mod used to validate this affects exactly how many gaps it closes.
- Card text and other CardGameDB- and TTS-mod-sourced fields (rules text,
  type, cost, stats, etc.) are © Games Workshop Limited / Fantasy Flight
  Games, reproduced here for reference and archival purposes only, the
  same way other fan-run LCG card databases operate. This repo is not
  affiliated with either company.

## Contributing

If you improve `EDITION_ALIASES`/`match_key` coverage or find another
archive source for the fully-missing expansions, a PR with the updated
script (and, ideally, a fresh `whi_detail_failures.csv` showing the
improvement) is welcome.
