# IPL Data Analysis

This project is an end-to-end data analysis on Indian Premier League cricket data. It is structured in three progressive layers — starting from raw NumPy array operations, moving into a full Pandas pipeline, and finally building a composite scoring model to predict the Orange Cap winner. The data used covers ball-by-ball delivery records and match-level metadata across multiple IPL seasons.

---

## Project Structure

```
IPL-DATA_ANALYSIS/
│
├── data/
│   ├── deliveries.csv
│   └── matches.csv
│
├── numpy/
│   └── numpy_main.ipynb
│
├── pandas/
│   ├── pandas_main.ipynb
│   ├── pandas_output/
│   └── df_final_processed.pkl
│
├── trend_analysis/
│   └── orange_cap_prediction.ipynb
│
└── README.md
```

---

## Datasets

The project uses two CSV files stored in the `data/` folder.

**deliveries.csv** contains ball-by-ball data — every delivery bowled across all matches, including the batter, bowler, runs scored, extras, wicket information, and over number.

**matches.csv** contains match-level information — teams playing, venue, city, season, winner, result margin, and the player of the match.

---

## NumPy Analysis

**`numpy/numpy_main.ipynb`**

This notebook does the entire analysis using raw NumPy arrays without any Pandas. The goal was to build a strong foundation in vectorised operations, masking, and array-based aggregation.

The tasks covered include:

- Loading both CSVs using `np.genfromtxt` and handling missing values
- Calculating total runs scored per match using boolean masking and `np.sum`
- Finding the top 5 batters by total runs using `np.bincount` and `np.argsort`
- Computing strike rates for every batter (runs divided by balls, multiplied by 100)
- Computing economy rates for every bowler (runs conceded per over)
- Analysing average runs scored in each of the 20 overs across all matches
- Counting total fours and sixes, and finding the team with the most boundaries
- Isolating death over (overs 16–20) performance by runs and team
- Identifying the single highest-scoring match in the dataset
- Breaking down runs scored per team per match

---

## Pandas Analysis

**`pandas/pandas_main.ipynb`**

This notebook builds a proper data pipeline from scratch — ingestion, cleaning, transformation, and analysis — all using Pandas.

### Data Ingestion

Both CSVs are loaded using `pd.read_csv`. The shapes, column names, and data types are inspected before any processing begins.

### Data Cleaning

The raw data had several issues that needed to be handled:

- Null values in columns like `player_dismissed`, `dismissal_kind`, and `fielder` were filled with `'none'`
- Team names were inconsistent across seasons — for example, `Delhi Daredevils` and `Delhi Capitals` refer to the same franchise. These were standardised across both files
- Null city values were resolved using a manual venue-to-city mapping for Dubai and Sharjah stadiums
- Match ID integrity was validated to confirm every delivery record links to a valid match

### Data Transformation

Before analysis, several new columns were engineered — total runs calculated independently, pure batting runs, and a ball number column that converts over and ball into a single sequential index. The two DataFrames were then merged into a single unified `df_final` on `match_id`.

### Analysis

- **Total runs per match** — grouped by match ID and summed
- **Runs per team per match** — multi-key groupby on match and batting team
- **Top 10 batters** — total runs scored across all matches, sorted descending
- **Strike rates** — computed on valid balls only (excluding wides, byes, leg byes, and penalties)
- **Top bowlers by economy** — runs conceded divided by overs, filtered for qualified bowlers
- **Most consistent batter** — average runs per match for players with at least 30 appearances

The processed DataFrame is saved as `df_final_processed.pkl` for use in the next notebook.

---

## Orange Cap Prediction

**`trend_analysis/orange_cap_prediction.ipynb`**

This notebook predicts the top Orange Cap candidates — the leading run-scorer of the season — using a composite scoring model built on the last three seasons of data.

### Feature Engineering

The model filters for batters who played at least 15 matches and computes the following per batter:

- **Batting Average** — total runs divided by dismissals
- **Strike Rate** — runs per ball, scaled to 100
- **Dot Ball Percentage** — proportion of balls faced where no run was scored
- **Powerplay Percentage** — share of balls faced in overs 1 to 6
- **Balls per Match** — a measure of how long a batter survives at the crease
- **Trajectory** — the difference in runs between the most recent season and the oldest of the three, capturing recent form

### Composite Score


orange_cap_score =
    (avg_runs_per_match × 14)     — base projection over a 14-match season
  + (batting_avg × 1.5)           — rewards consistency
  + (trajectory × 0.1)            — rewards improving recent form
  + (pp_percent × 0.5)            — rewards powerplay impact


The top 5 candidates are printed and visualised as a horizontal bar chart using Matplotlib.

---

## Tech Stack

- **Python 3**
- **NumPy** — array operations, masking, bincount
- **Pandas** — data cleaning, transformation, groupby analysis
- **Matplotlib** — visualisation
- **Jupyter Notebook** — interactive development environment

---

## Getting Started


Run the notebooks in this order:

1. `numpy/numpy_main.ipynb`
2. `pandas/pandas_main.ipynb`
3. `trend_analysis/orange_cap_prediction.ipynb`

> The Orange Cap notebook reads from `df_final_processed.pkl`, which is generated at the end of the Pandas notebook. Make sure to run that first.
