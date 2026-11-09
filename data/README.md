# Data

## About this data

**Source.** Three Wikimedia APIs: the MediaWiki Action API (people in Category:2024 deaths, and each article's length on the last revision before death), the Wikidata API (exact date of death), and the Wikimedia Pageviews API (daily views by human readers from 90 days before each death through the collection date). Every request uses an identifying User-Agent and pauses one second between requests.

**Collected.** November 8, 2026, using `notebooks/02_collect_and_clean.ipynb`.

**Records.** Category:2024 deaths held 11,939 articles. I randomly sampled 300 (seed 4617). After cleaning, 133 people remain: those with an exact 2024 death date, an article that existed before they died, and a peak of at least 500 daily views in the 30 days after death.

**What is in the repo.** A 10-person sample of the cleaned data, in `sample/`.

**What I left out and why.** The raw API responses and full cleaned tables are not committed. The data is openly licensed and contains no reader information, so this is about size, not permission: the full tables are several megabytes, and anyone can rebuild them by running the notebook.

## people_sample.csv

One row per person.

| Field | Meaning |
|---|---|
| title | Title of the person's English Wikipedia article |
| wikidata_id | The person's Wikidata ID |
| death_date | Date of death from Wikidata, exact to the day |
| length_before_death | Article size in bytes on the last revision before the person died |
| baseline_views | Median daily views in the 90 days before death |
| peak_views | Highest daily views in the 30 days starting on the day of death |
| peak_date | Date of that highest day |
| length_group | Short, medium, or long, splitting articles into equal thirds by length |

## pageviews_daily_sample.csv

One row per person per day, covering 14 days before to 60 days after each death.

| Field | Meaning |
|---|---|
| title | Title of the person's English Wikipedia article |
| date | Calendar date (UTC) |
| days_since_death | Days relative to death; 0 is the day of death, negative is before |
| views | Views by human readers that day. Days the API did not report are filled with 0 |
