# Denver classroom dataset: sources and preparation

## Source and attribution

Provider: **Inside Airbnb**. Source release label: **June 30, 2026**. Downloaded September 8, 2026.

- [Source download page](https://insideairbnb.com/get-the-data/)
- [Detailed listings download](https://data.insideairbnb.com/united-states/co/denver/2026-06-30/data/listings.csv.gz)
- [Detailed written reviews download](https://data.insideairbnb.com/united-states/co/denver/2026-06-30/data/reviews.csv.gz)
- [Collection assumptions](https://insideairbnb.com/data-assumptions/)
- [Creative Commons Attribution 4.0 license](https://creativecommons.org/licenses/by/4.0/)

The provider's download page identifies the release as June 30. The listing file's `last_scraped` values include June 30 and July 2, 2026. These are a released snapshot, not a live feed.

The raw archives are stored in `raw/`. The [provenance record](provenance.json) includes exact URLs, file sizes, SHA-256 checksums, and preparation counts. The preparation code is [prepare_denver_data.py](../../prepare_denver_data.py).

## Listing table

`denver_listings.csv` retains **all 4,939 rows** of the detailed listing file and selects **16 columns**. Rows have not been sampled or filtered by rating, price, or neighborhood. Missing values and extreme prices remain available for teaching EDA. Prices retain the source currency formatting so the notebook can demonstrate conversion.

The original summary listing file in `CLASS` has 4,949 rows. It differs from the detailed source by 10 listings. The classroom table uses only the detailed source and does not combine these releases' rows.

| Classroom field | Source field | Meaning |
|---|---|---|
| `listing_id` | `id` | Unique listing identifier |
| `name` | `name` | Listing title |
| `neighbourhood` | `neighbourhood_cleansed` | Source's geographically assigned neighborhood |
| `room_type` | `room_type` | Listing category |
| `accommodates` | `accommodates` | Advertised guest capacity |
| `bedrooms` | `bedrooms` | Reported bedrooms, sometimes missing |
| `price` | `price` | Listed nightly price text in USD |
| `minimum_nights` | `minimum_nights` | Listed minimum stay |
| `number_of_reviews` | `number_of_reviews` | Reported listing review count |
| `availability_365` | `availability_365` | Listed availability in the coming 365 days |
| `overall_rating` | `review_scores_rating` | Aggregate overall listing score |
| `cleanliness_rating` | `review_scores_cleanliness` | Aggregate cleanliness score |
| `location_rating` | `review_scores_location` | Aggregate location score |
| `value_rating` | `review_scores_value` | Aggregate value score |
| `latitude` | `latitude` | Published listing latitude |
| `longitude` | `longitude` | Published listing longitude |

The table represents 77 neighborhoods. There are 4,301 observed overall ratings and 638 missing ones. The observed rating scale is 1–5. There are 688 missing prices. The notebook computes statistics from observed values and reports counts.

## Written-review sample

The full detailed review source has **354,741** rows, dated August 28, 2008 through June 30, 2026. Unlike the summary review CSV, it contains a `comments` field and a unique review ID.

To prepare a manageable teaching file:

1. Restrict the date to **July 1, 2025 through June 30, 2026**, inclusive.
2. Require nonblank comments and a listing ID found in the detailed listing table. Across the entire raw review file, 95 records reference listings absent from the detailed listing source.
3. Sample **10,000** reviews without replacement from the **62,341** eligible records using pandas `sample(random_state=42)`.
4. Keep the review ID, listing ID, date, and comments. Rename `id` to `review_id` and `comments` to `review_text`.
5. Decode HTML entities, remove HTML tags, and normalize whitespace. Do not translate, correct spelling, filter sentiment, or remove non-English reviews.
6. Sort the sample by date and review ID. Omit reviewer names and reviewer IDs from the classroom file.

The result contains reviews for **2,459 listings**. The actual sampled dates run from July 1, 2025 to June 29, 2026; a random sample need not include a record from the final eligible day. Every sampled review matches exactly one classroom listing, and the review IDs are unique.

## Interpretation boundaries

- Ratings belong to listings and summarize their review histories; they are not individual review stars and are not restricted to the sampled recent year.
- A mean of listing ratings weights each rated listing equally. Joining ratings onto reviews and averaging there would instead give more weight to listings with more sampled reviews.
- The text sample represents sampled review records, not an equal number of reviews per listing. Popular or frequently reviewed listings can contribute more text. Long reviews contribute more word occurrences.
- The 10,000-row sample is not the complete record of reviews, bookings, or stays. Review counts do not directly measure occupancy or demand.
- Prices are listed snapshot values rather than full checkout costs. Missing prices and unusually high values deserve inspection, not silent replacement.
- Counts of listings by neighborhood are not density measures. Neighborhood areas differ. Published coordinates are not exact verified addresses.
- Word clouds count processed terms. They do not measure sentiment, importance, or the prevalence of a qualitative theme. The English stopword lists used in the lesson do not handle every language equally.
- K-means labels are exploratory groups based on the selected representation and parameters. They need interpretation and are not verified categories or causal findings.

The files are supplied for a grounded teaching exercise about data inspection and careful interpretation. A good student conclusion states which rows and fields support it and what remains uncertain.

## Carless notebook's analysis population

The notebook uses the classroom files above and makes additional in-memory selections for a demo trip. Its defaults are an entire home/apartment, capacity for at least two guests, and a listed minimum stay of no more than three nights. This selects **2,033 listings across 75 neighborhoods**. Price and rating summaries refer to those listings, counting observed values separately from all rows.

The inner join retains **7,639 sampled reviews** whose listing IDs are in that selection. All keyword percentages use this selected review population. A review is counted once per topic, regardless of repeated words; overlapping topics are allowed. The default broad walking search matches **1,505 reviews**. Highland contributes 160 matching listings and 550 sampled reviews, of which 160 match the walking search (approximately 29.1%).

Each area profile combines listing-level prices/ratings with review-level counts and mention percentages. It never estimates average listing prices or scores by averaging repeated rows after the review join. Missing review percentages remain unknown when a neighborhood has no selected reviews. Evidence thresholds and the area-median price screen are teaching parameters, not a validated car-free suitability model.

Search patterns are defined visibly in the notebook. English wording, false positives, negative statements, missed synonyms, and uneven review coverage all affect interpretation. A location rating and a walking mention are starting signals; route feasibility and destination-specific travel conditions are not supplied by these files. Changing the trip filters changes both the listing and review populations and requires rerunning the downstream cells.
