# Nobel Prize Laureates: Data Cleaning Project

Cleaning a raw public dataset of Nobel Prize laureates so it is ready for analysis.

- **Source:** [TidyTuesday 2019-05-14, `nobel_winners.csv`](https://github.com/rfordatascience/tidytuesday/tree/master/data/2019/2019-05-14) (original data from the Nobel Prize API)
- **Tool:** Python (pandas), one reproducible script: `clean_nobel.py`
- **Result:** 969 rows x 18 columns (raw) -> 911 rows x 22 columns (clean)

## Repository layout

```
data/raw/nobel_winners.csv            original, untouched
data/clean/nobel_winners_clean.csv    cleaned dataset
data/clean/country_mapping.csv        every country value changed (original -> cleaned)
data/clean/cleaning_log.csv           issue-by-issue log with counts
clean_nobel.py                        the cleaning script
```

Run it yourself: `pip install pandas && python clean_nobel.py`

## Issues found and what I did

### 1. Missing values
| Column | Missing | Why | Action |
|---|---|---|---|
| `motivation` | 88 | All Peace prizes (no citation published for early ones) | Left blank |
| `birth_*`, `gender` | 26-30 | Organizations (e.g. Red Cross) have no birth data | Left blank, not applicable |
| `death_*` | 318 | Laureate is still alive | Left blank and added `is_living` flag |
| `organization_*` | 244-246 | Laureate had no institutional affiliation (common in Literature and Peace) | Left blank |

No values were imputed. Filling these with guesses would have invented information.

### 2. Duplicate records
- **Fully identical rows:** 0 found.
- **Hidden duplicates:** 58 rows repeated the same laureate for the same prize, differing only in the affiliation columns (one row per university). These were collapsed to **one row per laureate per prize**. The first listed affiliation stays in the `organization_*` columns and any extras are kept in the new `other_affiliations` column, so no information is lost.
- A laureate who won twice (e.g. Marie Curie) correctly keeps two rows.

### 3. Incorrect data types
- `birth_date`, `death_date`: text -> proper dates (all parsed, none failed).
- `prize_share`: text like `"1/3"` -> added numeric `prize_share_fraction` (0.3333). Original text kept.
- `category`, `laureate_type`, `gender`: treated as categorical.
- Added `age_at_award` (prize year minus birth year) as a ready-to-use analysis field.

### 4. Inconsistent values
- **HTML entities** in 14 cells (e.g. `W&uuml;rttemberg`, `Unit&eacute; Mixte`) decoded to real characters.
- **`motivation`** was wrapped in literal quote marks in 881 rows; removed.
- **Mislabeled type:** 4 people were tagged `Organization` (Le Duc Tho, Mother Teresa, the 14th Dalai Lama, Aung San Suu Kyi) even though they have gender and birth dates. Relabeled `Individual`.
- **Country names mixed historical and modern forms**, e.g. `Prussia (Poland)`, `West Germany (Germany)`, `Federal Republic of Germany`, `Austria-Hungary (Hungary)`. I mapped them to the present-day country (the value in parentheses). Distinct values: `birth_country` 121 -> 78, `death_country` 50 -> 46, `organization_country` 29 -> 27. The full 67-entry mapping is in `country_mapping.csv`.

## Judgment calls and limitations
- Present-day country names make country-level counts comparable over time, but they discard the historical name (that is what `data/raw` is for).
- `Czechoslovakia` and `Union of Soviet Socialist Republics` appear **without** a modern country in the source, and each has several successor states, so I left them as is rather than guess.
- `Scotland`, `Northern Ireland` and similar constituent countries were kept as the source lists them.
- One laureate has a death country but no death date; left as is because I cannot verify it.
- Validation checks in the script confirm: no duplicate laureate-prize pairs, no death date before birth date, and all prize fractions between 0 and 1.
