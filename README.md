# DS@GT Athletics Scrapers

Python helpers for collecting and cleaning men's college basketball data for [Data Science at Georgia Tech (DS@GT)](https://datasciencegt.org/). This repo is the club's working copy of the 2023 athletics scraper package: it talks to the same NCAA Casablanca JSON endpoints and Hoop-Math HTML pages the original project targeted, and returns pandas DataFrames you can clean, join, and analyze.

**Who it is for:** incoming DS@GT members, a future Sports Analytics project lead, and anyone rebuilding a Fall 2026 sports pipeline on top of this code. It is a data-collection library, not a finished analysis project and not a live stats dashboard.

This is [DataScience-GT/DSGT-AthleticsScrapers](https://github.com/DataScience-GT/DSGT-AthleticsScrapers), forked from [Gerald-Lu/DSGT-AthleticsScrapers](https://github.com/Gerald-Lu/DSGT-AthleticsScrapers) on **2026-08-28**.

---

## Current status (Fall 2026)

| Fact | Detail |
| --- | --- |
| This fork | Created in the `DataScience-GT` org on 2026-08-28 by Aamogh Sawant. Code is unchanged from upstream at the time of the fork. |
| Upstream | Last public commit on `Gerald-Lu/DSGT-AthleticsScrapers` is **2023-11-24** (`636972a`, package version `0.0.2`). |
| Club project listing | [datasciencegt.org](https://datasciencegt.org/) lists **Sports Analytics as Closed** (late August 2026). The archive page credits a Casper Guo–era "Sports Analysis Project." |
| Fall 2026 sports lead | **Not confirmed.** Recruiting a lead is an open work item (see [Next steps](#next-steps-for-fall-2026)). |
| This README | Rewritten for incoming members. It does **not** claim the scrapers still work against today's sites. |

Do not treat the 2023 README examples as current Georgia Tech or NCAA results. Those strings (`6049153`, `2022/11/10`, `GeorgiaTech2023`) are usage examples from the original authors, not verified live data.

---

## How this repo relates to the others

| Repo | What it actually contains | Use it for |
| --- | --- | --- |
| **This repo** — [DataScience-GT/DSGT-AthleticsScrapers](https://github.com/DataScience-GT/DSGT-AthleticsScrapers) | The Python package: four scraper functions, packaging metadata, MIT license. | Collect / clean athletics data. Club work starts here. |
| [Gerald-Lu/DSGT-AthleticsScrapers](https://github.com/Gerald-Lu/DSGT-AthleticsScrapers) | Upstream source (DSGT Athletics Sub Team 6, 2023). Same tree as this fork as of 2026-08-28. | History and original packaging. Do not assume it will receive new commits. |
| [DataScience-GT/FA24-Sports-Analysis](https://github.com/DataScience-GT/FA24-Sports-Analysis) | **Empty Fall 2024 shell.** Created 2024-09-24; last push 2024-10-05. Files on `main`: `README.md` (`# FA24-Sports-Analysis` only) and `.gitignore`. No scrapers, notebooks, or data. | Historical pointer only. **Do not edit that repo** for this revival. Put analysis notebooks in a *new* follow-on repo if needed. |

**Hypothesis:** FA24-Sports-Analysis was stood up as a Casper Guo–era analysis home and never received code. This scraper repo is the only org repo with actual athletics collection code.

---

## What the package does

`DSGTAthleticsScrapers` (version `0.0.2` in `setup.cfg`) exports four functions from [`DSGTAthleticsScrapers/__init__.py`](DSGTAthleticsScrapers/__init__.py):

```text
NCAA_game_id_scraper(date)          -> DataFrame
NCAA_box_scraper(game_id, team_name) -> DataFrame
NCAA_pbp_scraper(game_id)           -> DataFrame
HoopMath_scraper(team_slug)         -> (offense DataFrame, defense DataFrame)
```

There is no CLI, no notebooks, and no test suite in this tree (tests were removed in commit `4546042`). Typical flow:

```text
date string  --NCAA_game_id_scraper-->  Game ID + Teams
game ID      --NCAA_box_scraper------>  one team's box score
game ID      --NCAA_pbp_scraper------>  play-by-play (uses box score names)
team slug    --HoopMath_scraper------>  offensive + defensive transition tables
```

`NCAA_pbp_scraper` also fetches the box score JSON so it can attach player names to each play. Hoop-Math is independent of the NCAA helpers.

### Legal / terms-of-use caution

- Scrape **only** the sources this package already targets: NCAA Casablanca JSON under `data.ncaa.com` for **men's Division I basketball**, and Hoop-Math team pages matching `https://hoop-math.com/{slug}.php`.
- Do not add new sites, sports, or endpoints unless that work is explicitly scoped as future work and reviewed against those sites' terms.
- Be respectful of rate limits. The current code has **no** delays, retries, or custom User-Agent. Do not run tight loops against live endpoints.
- NCAA.com / Hoop-Math terms can change. If a source forbids automated access, stop and ask leadership before continuing.
- Upstream authors already noted that some NCAA box scores and play-by-play feeds are incomplete. Treat missing or oddly worded plays as a data-quality issue, not as a win/loss fact.

---

## Setup

Declared in [`setup.cfg`](setup.cfg):

- Python `>= 3.7`
- `requests>=2.26.0`
- `beautifulsoup4>=4.10.0`
- `pandas>=1.3.3`

**Hypothesis:** 3.7 is past end-of-life. Fall 2026 work should pin a current CPython (for example 3.11 or 3.12) after someone actually runs the functions.

### Install from this repo (preferred for club work)

```bash
git clone https://github.com/DataScience-GT/DSGT-AthleticsScrapers.git
cd DSGT-AthleticsScrapers
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -e .
```

Editable install reads `pyproject.toml` + `setup.cfg` and pulls the three dependencies above.

### PyPI package

`pip install DSGTAthleticsScrapers` still resolves on PyPI as **0.0.1 / 0.0.2** (the 2023 upload). That is the *published upstream wheel*, not a rebuild of this org fork. Use the clone + `pip install -e .` path until the club decides whether to republish.

### Import and call

```python
from DSGTAthleticsScrapers import (
    NCAA_game_id_scraper,
    NCAA_box_scraper,
    NCAA_pbp_scraper,
    HoopMath_scraper,
)

# Men's D1 scoreboard for one calendar day (YYYY/MM/DD).
# Example date is from the 2023 README, not a claim about that day's results.
game_ids = NCAA_game_id_scraper("2022/11/10")

# Box score for one team in one game. team_name is matched to NCAA shortName
# (case-insensitive), e.g. the string that appears as shortName in boxscore.json.
box = NCAA_box_scraper(6049153, "Georgia Tech")

# Play-by-play for one game id (from ncaa.com/game/<id>/... or the Game ID column).
pbp = NCAA_pbp_scraper(6049153)

# Hoop-Math slug is the path segment before .php, e.g. hoop-math.com/GeorgiaTech2023.php
offense, defense = HoopMath_scraper("GeorgiaTech2023")
```

If a call raises (HTTP error, missing JSON keys, `None` team match, missing HTML tables), record the exception. That is expected verification work, not a bug in this README.

---

## Repo layout

```text
DSGTAthleticsScrapers/
  __init__.py            # re-exports the four public functions
  NCAAScrapers.py        # NCAA_game_id_scraper, NCAA_box_scraper, NCAA_pbp_scraper
  HoopMathScraper.py     # HoopMath_scraper
setup.cfg                # package name, 0.0.2, dependencies, python_requires
pyproject.toml           # setuptools build backend
LICENSE                  # MIT, 2023, Athletics Sub Team 6
README.md                # this file
.gitignore               # currently only .DS_Store
build/, dist/, *.egg-info/   # leftover 2023 packaging artifacts (committed upstream)
```

Original authors listed in `setup.cfg` / `LICENSE`: **Gerald Lu, Akshar Ravichandran, Om Rajpal** (DSGT Athletics Sub Team 6). Contact on the old README was `glu49@gatech.edu`; use club contacts below for Fall 2026 work.

---

## Scrapers and output columns

Columns below are taken from the code. They are the DataFrame schema the functions *try* to build, not a guarantee that a live request still returns matching JSON/HTML.

### `NCAA_game_id_scraper(date)`

- **File:** `DSGTAthleticsScrapers/NCAAScrapers.py`
- **Request:** `GET https://data.ncaa.com/casablanca/scoreboard/basketball-men/d1/{date}/scoreboard.json`
- **Input:** `date` as `YYYY/MM/DD` (same shape as the NCAA scoreboard path).
- **Output columns:** `Game ID`, `Teams`
- **Notes:** Game ID is `game.url` with the first six characters stripped. `Teams` is `game.title`. Hardcoded `Cookie: akacd_ems=...` header is sent on every NCAA request (see [Known issues](#known-issues-from-reading-the-code)).

### `NCAA_box_scraper(game_id, team_name)`

- **Request:** `GET https://data.ncaa.com/casablanca/game/{game_id}/boxscore.json`
- **Input:** numeric/string game id; `team_name` compared to `meta.teams[].shortName` (case-insensitive).
- **Output columns:** `Name`, `Position`, `Minutes Played`, `Field Goals Made`, `Three Pointers Made`, `Free Throws Made`, `Total Rebounds`, `Offensive Rebounds`, `Assists`, `Personal Fouls`, `Steals`, `Turnovers`, `Blocked Shots`, `Points`
- **Notes:** One row per player, plus a final `Total` row from `playerTotals`. If `shortName` does not match, `chosen_team` returns `None` and `write_df` will fail.

### `NCAA_pbp_scraper(game_id)`

- **Requests:** `.../game/{id}/pbp.json` and `.../game/{id}/boxscore.json`
- **Output columns:** `Time Left`, `Score`, `Play Type`, `Player 1 Involved`, `Player 2 Involved`, `Player 3 involved`, `Play Result`
- **Notes:** Concatenates period 0, period 1, then any overtime periods. Assumes at least two periods exist. Play Type is the first substring hit from a hardcoded English list (`Jumper`, `Layup`, `timeout`, `time out`, `TIMEOUT`, …). Unrecognized wording leaves Play Type empty — the original README already called this out. Player slots are filled by matching box-score first/last names inside the play text; unused slots are `N/A`.

### `HoopMath_scraper(team_slug)`

- **File:** `DSGTAthleticsScrapers/HoopMathScraper.py`
- **Request:** `GET https://hoop-math.com/{team_slug}.php` (BeautifulSoup)
- **Tables:** `#TransOTable1` (offense), `#TransDTable1` (defense)
- **Output:** two DataFrames whose **headers are scraped from the page** (not hardcoded). Rows are packed by walking every 9 `<td>` cells.
- **Notes:** Non-200 responses raise `Exception("Failed to load page.")`. Missing tables become `None` and then fail in `write_df`.

The original examples used slug `GeorgiaTech2023` and game `6049153` only as parameter illustrations.

---

## Known issues (from reading the code)

These are observations about the 2023 source. They are **not** a live test log.

1. **Stale cookie (hypothesis).** All three NCAA functions send a fixed Akamai cookie (`akacd_ems=1695693092~...`). The numeric prefix looks like a ~2023 epoch. If NCAA now requires a fresh cookie or no cookie, requests may fail or return HTML/errors instead of JSON.
2. **No robustness.** No timeout, retry, status-code check (NCAA path), or rate limit. NCAA helpers call `response.json()` unconditionally.
3. **Brittle NCAA JSON shape.** Game-id code expects `games[].game.url` and `title`. Box/PBP code expects `meta.teams`, `teams[].playerStats` / `playerTotals`, and `periods[].playStats`. Any rename breaks the function.
4. **PBP assumes two regulation periods.** `pbp_data['periods'][0]` and `[1]` are always read. A game with missing PBP will throw.
5. **Hoop-Math layout (hypothesis).** Table ids and a 9-cell row width are hardcoded. A site redesign would break parsing even if the page still loads.
6. **Dependencies are lower-bounded only.** No upper pins; no lockfile. `python_requires >= 3.7` is the 2023 declaration.
7. **Packaging clutter.** `build/`, `dist/`, and `DSGTAthleticsScrapers.egg-info/` are committed. `.gitignore` does not ignore them.
8. **PyPI vs this fork.** Installing from PyPI does not track commits on `DataScience-GT/DSGT-AthleticsScrapers`.

---

## Next steps for Fall 2026

Concrete work streams. Label anything unverified as a hypothesis when you file issues.

1. **Verify each scraper against current sources.** Run the four functions once (one date, one game id, one Hoop-Math slug the original project already used). Write down HTTP status, exception, and whether the DataFrame columns still match this README. Do not invent new target sites. Do not commit large dumps.
2. **Pin dependencies / modernize Python if needed.** Add a lockfile or pinned `install_requires`, pick a supported CPython, and only then bump `python_requires`. **Hypothesis:** the Akamai cookie and unpinned pandas/requests are the first things that will fail on a fresh machine.
3. **Document output schema and sample data.** Keep schema in this README (or a short `docs/schema.md`). Check in **tiny** fixtures (a few rows of anonymized/redacted sample), never season-sized CSVs.
4. **Decide whether to rebuild a Fall 2026 Sports Analytics project on this pipeline.** Club site currently marks the project Closed. That decision sits with the President and Director of Projects after scrapers are shown to work (or after a scoped rewrite).
5. **Recruit a project lead and a meeting time after 6:30 PM ET.** No Fall 2026 sports lead is confirmed as of late August 2026.
6. **Optional — analysis notebooks in a follow-on repo.** Do not turn `FA24-Sports-Analysis` into a second empty README, and do not add a fake analysis tree here. If the club wants notebooks, create a new repo once this pipeline is verified.

Suggested first issues: "NCAA game-id request 2026 smoke test", "Hoop-Math table ids still present?", "Pin Python + dependencies", "Ignore dist/build/egg-info".

---

## How to join and contribute

**Club**

- Site / join: [datasciencegt.org](https://datasciencegt.org/)
- Club email: [hello@datasciencegt.org](mailto:hello@datasciencegt.org)
- President (2026–2027): Aamogh Sawant
- Director of Projects: Samantha Forero — [sforeror3@gatech.edu](mailto:sforeror3@gatech.edu)

Email Samantha or `hello@datasciencegt.org` if you want to help verify scrapers, lead the sports project, or propose a meeting slot after 6:30 PM ET.

**Code**

1. Fork or branch from [DataScience-GT/DSGT-AthleticsScrapers](https://github.com/DataScience-GT/DSGT-AthleticsScrapers) (`main`).
2. Keep changes small: one scraper fix, one docs/schema update, or one dependency pin per PR.
3. Do not commit scraped season dumps. Do not add new scrape targets without a written ToS check.
4. Open a pull request against `main` and describe what you ran and what broke.

License remains MIT (2023 Athletics Sub Team 6). Keep that notice when you copy or republish the package.
