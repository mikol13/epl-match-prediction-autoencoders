# Data requirements

The notebook expects these local CSV files:

- `match.csv` — fixture identifiers, dates, team identifiers, and final scores.
- `league.csv` — competition IDs and names; the notebook uses it to select Premier League seasons.
- `league_season.csv` — links each season to its competition and provides season dates.
- `vaastav_stats.csv` — player-by-match Fantasy Premier League statistics, including match ID and home/away indicator.

Place the files under `data/raw/` or in the project root. The notebook checks both locations. Raw datasets are excluded from version control; obtain and redistribute them only in accordance with their original source terms. The reported source sample covers Premier League seasons 2017/18 through 2023/24.

The reference article uses FBref and ClubElo data. These files are an alternative Vaastav/FPL-based input and should not be described as the paper's original dataset.
