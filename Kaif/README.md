# Netflix Movies & TV Shows — EDA
### Gray Interface '26 | Task 1 | Kaif

---

## Project Overview

This project demonstrates mastery over Data Wrangling and Exploratory Data Analysis on the Netflix Movies and TV Shows dataset. There is zero model training — the entire focus is on cleaning messy raw data and extracting meaningful visual insights from it.

**Dataset:** [Netflix Movies and TV Shows — Kaggle](https://www.kaggle.com/datasets/shivamb/netflix-shows/data)

The dataset was loaded programmatically using the `kagglehub` library inside Google Colab — no manual CSV downloading or GUI uploading was used.

---

## Part 1: Data Engineering Pipeline

### The Duration Fix

The `duration` column stored values as plain strings like `"90 min"` for movies and `"3 Seasons"` for TV shows, which cannot be used numerically. Two separate numeric columns were created — `movie_duration(in minutes)` and `tv_show_seasons` — by applying a `lambda` function to every row.

The `lambda` function uses `split()` to break the string by whitespace, picks the first element (the number), and converts it to an integer. A conditional check on `'min'` or `'Season'` ensures each value goes into the correct column. Rows that don't match return `None`. The original `duration` column was then dropped since it was no longer needed.

---

### Unnesting Arrays

Columns like `cast`, `director`, `country`, and `listed_in` stored multiple values as comma-separated strings — for example, `"Tom Hanks, Robin Wright"`. Treating these as a single string makes it impossible to count individual actors or genres accurately.

The `.str.split(',')` method was applied to each of these columns, converting every cell from a plain string into a Python list of individual values. This allows downstream operations like `groupby` and `value_counts` to treat each actor, country, or genre as a separate entity.

---

### Datetime Parsing

The `date_added` column was stored as a raw string like `"January 1, 2020"`. The `pd.to_datetime()` function converted it into a proper datetime object, with `errors='coerce'` handling any malformed or missing dates gracefully by converting them to `NaT` instead of raising an error.

Once parsed, the `.dt` accessor was used to extract three new columns — `year_added`, `month_added`, and `day_of_week_added` — enabling all temporal grouping and trend analysis.

---

## Part 2: Visualizations

### Temporal Trends — Line Chart

**Question: How has Netflix's focus shifted between Movies and TV Shows over the last decade?**

The data was grouped by `year_added` and `type` using `groupby`, counting how many titles of each type were added per year. This was then pivoted so Movies and TV Shows became separate columns and plotted as a multi-line chart.

**Outcome:** Netflix initially added far more Movies than TV Shows. From 2016 to 2019, additions of both types accelerated sharply, with Movies consistently outnumbering TV Shows. Post-2019, the pace slowed across both categories — likely reflecting content licensing strategy changes. Movies have always been the dominant content type by volume throughout the decade.

---

### Monthly Releases — Bar Chart

**Question: Which months are most popular for new releases?**

The data was grouped by `month_added` and `type`, counting releases per month for both Movies and TV Shows. A bar chart with `hue='type'` shows both categories side by side for each month. The most popular month was identified using `value_counts().idxmax()`.

**Outcome:** July (Month 7) is the single most popular month for new content additions on Netflix. There is also a notable spike towards the end of the year in October–December, aligned with holiday season viewing demand. January and February are the quietest months for new releases.

---

### Country-Year Heatmap

**Question: Which countries collaborate the most on Netflix content, and how has this changed over time?**

The data was grouped by `country` and `year_added`, then filtered to the top 10 most prolific countries. This was pivoted into a matrix with years as rows and countries as columns, and plotted as a heatmap with the `YlGnBu` color palette and cell annotations showing exact counts.

**Outcome:** The United States dominates Netflix content production across all years, shown by significantly darker cells. India is the second-largest contributor, particularly from 2018 onwards, reflecting Netflix's investment in Indian content. The UK is a consistent third. Most countries peaked in contributions between 2018 and 2020. Japan and South Korea show growing presence in recent years, consistent with the global rise of anime and K-dramas.

---

### Actor-Director Duos — Bar Chart

**Question: Who are the most frequent actor-director collaboration pairs on Netflix?**

The data was grouped by `cast` and `director` together using `groupby`, counted, sorted in descending order, and the top 5 pairs were selected. A horizontal bar chart with `hue='director'` color-codes each bar by director, making it easy to distinguish which pairs worked together most.

**Outcome:** The chart identifies the top 5 most recurring creative partnerships on Netflix. These frequent duos represent directors who consistently cast the same actors across multiple projects — a sign of strong long-term creative relationships on the platform.

---

### Genre Distribution for Top 10 Actors — Bar Chart

**Question: How does the genre distribution look for the top 10 actors on Netflix?**

The top 10 most frequently appearing actors were identified using `value_counts().nlargest(10)`. The DataFrame was then filtered to only their titles, grouped by actor and genre, and plotted as a horizontal bar chart with genres as the color hue.

**Outcome:** The top actors on Netflix are heavily concentrated in Stand-Up Comedy and Documentaries rather than traditional films. This reveals that Netflix's most frequently appearing "actors" are often comedians with multiple stand-up specials. Genre specialization is strong — actors dominating one genre rarely cross over significantly into others.

---

### Most Common Genre Combinations — Bar Chart

**Question: What are the most common combinations of genres used to categorize Netflix content?**

A `lambda` function was applied to the `listed_in` column: it splits each genre string, sorts the individual genres alphabetically, then rejoins them. The sorting step ensures that `"Comedy, Drama"` and `"Drama, Comedy"` are treated as the same combination and not double-counted. The result was grouped and the top 10 combinations were plotted.

**Outcome:** Netflix relies heavily on broad, stacked genre tags to maximize discoverability. The most common combinations are "International Movies + Dramas" and "International Movies + Comedies", reflecting the platform's large international content library. Stand-alone "Documentaries" also appear frequently as single-genre titles, confirming their strong standalone identity on the platform.

---

## Summary of Key Findings

- Most popular month for new releases: **July**
- Dominant content type: **Movies** consistently outnumber TV Shows
- Top content-producing country: **United States**, followed by India and UK
- Peak content addition years: **2018–2020**
- Top appearing actors are mostly **stand-up comedians** with multiple specials
- Most common genre combination: **International Movies + Dramas**
