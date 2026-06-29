# Netflix Data Wrangling (EDA Task) 
**Submitted by:** [Kirti, roll no. 2560003]  
**Team:** task one (Gray Interface '26)

---

## 1. Introduction
The goal of this assignment was to master raw data processing and Exploratory Data Analysis (EDA) using the Netflix Movies and TV Shows dataset. As Per the project requirements, I wrote the script using Google Colab.

---

## 2. Data Engineering Methodology 

While diving into the raw dataset, I noticed several structural anomalies that would completely break standard analytical functions if left unaddressed. Here is how and why I engineered the data before plotting:

### Programmatic Ingestion via `kagglehub`
* I initialized the runtime ecosystem by fetching the `shivamb/netflix-shows` archive via `kagglehub.dataset_download()`.
* Because downloading files via the local browser and reuploading them through the Colab GUI violates professional reproducibility practices. Doing it programmatically means anyone can pull my repository and replicate the entire pipeline instantly.

### Parsing the Mixed 'Duration' Attribute
* The original `duration` column is stored as a string field mixed with text labels like `"90 min"` or `"3 Seasons"`. I used standard Python lambda operations and condition checks to safely split these strings. This allowed me to isolate two clean, numeric parameters: `duration_minutes` (numerical type) and `duration_seasons` (integer type).
* This was done because we cannot calculate statistical insights (like mean, median, or variance) on text strings. Separating them allows us to mathematically analyze how feature film lengths and show life cycles behave across the platform.

### Standardizing Calendar Timestamps
* The `date_added` feature contained whitespace issues and non-standard calendar names (e.g., `" August 14, 2020"`). I stripped out the whitespace padding and parsed it into standard timestamp structures using `pd.to_datetime()` with the format mapping `%B %d, %Y`. From this clean column, I pulled out specific granular fields: `added_year`, `added_month`, and `added_day_of_week`.
* Raw text strings representing dates cannot be sorted chronologically by Pandas. Converting them to true timestamp parameters unlocks powerful time-series operations.

### Vector Explosions for Multi-Value Arrays
* Categorical columns like `cast`, `director`, `country`, and `listed_in` frequently package multiple entities into a single, comma-separated string block (e.g., `"Director A, Director B"`). To solve this, I first handled missing information by filling null objects with `'Unknown'`. Then, I converted the strings into lists with `.str.split(', ')` and completely flattened the observations into individual data entries using the `.explode()` framework.
* If multiple actors or directors remain chained in a single cell, Pandas aggregates that combined string as its own isolated, unique entity. This completely skews value counts.

---

## 3. Visualizations 

Every plot generated inside my Jupyter Notebook was designed with clear titles, structured axes, and specific color choices to make interpreting the trends straightforward.

### 1: Content Type Additions Trend (2010 - 2021)
* Looking at this timeline, a major strategic shift stands out. Up until 2016, Netflix grew its library of Movies and TV Shows at almost identical rates. However, post-2016, Movie additions surged exponentially, vastly outpacing episodic TV Shows. 
* This marks the exact era where major legacy networks began planning their own streaming alternatives (like Disney+ and HBO Max). Netflix anticipated losing streaming licenses for long-running network shows and aggressively pivoted to purchasing self-contained feature films to bulk up its catalog.

### 2: Top 10 Directors
* Using the array explosion method to count every individual filmmaker credit accurately, this bar chart shows exactly who directs the most content on the platform. 
* The data shows that the highest-ranking directors aren't necessarily the most famous Hollywood names, but rather highly prolific international and independent creators. This points to long-term licensing deals and direct partnerships that Netflix established to keep a steady flow of exclusive content coming in.

### 3: Top Countries
* The bar chart breaks down the origin countries responsible for content creation. While the United States predictably leads the pack by a wide margin, India and the United Kingdom stand out as massive baseline contributors.
* This visual highlights Netflix's aggressive international expansion model. To capture subscribers outside of North America, they established massive local production hubs in regions with massive, established domestic entertainment industries.

### 4: Genre & Text
* **Part A: Genre Combinations:** Looking at the top half of the dashboard, standalone categorical pairings dominate the catalog. Broad classifications like *"Dramas, International Movies"* and *"Comedies, International Movies"* appear far more frequently than highly specific, niche genres. 
  * *Insight:* Netflix intentionally packages content into broad, universally appealing buckets to make sure titles travel well across different regional borders.
* **Part B: Word Frequencies (Horror vs. Comedy):** The comparative word clouds display clear, stark differences in vocabulary choices within descriptions.
  * *Horror:* Highly focused on visceral and high-stakes environmental nouns like *"killer"*, *"mysterious"*, *"death"*, *"house"*, and *"town"*.
  * *Comedy:* Strongly relies on dynamic relationship tags and lighter situational words like *"life"*, *"friend"*, *"love"*, *"family"*, and *"woman"*.
  * *The Coolest Catch:* Interestingly, the word **"find"** is a dominant anchor word in *both* clouds. This highlights a fundamental rule in screenwriting across genres: the plot of almost every film or show revolves around characters on a quest to locate or figure something out, whether they are running from a monster or looking for love.

---
