# NASA Exoplanet Archive — Data Science Lab Notes

> **How to read this file in VS Code:** Press **`Ctrl + Shift + V`** to switch from raw text to the clean, formatted Reader View!

---

## PART I: Step-by-Step Build Manual

### Workspace & Git Setup (Before Coding)

1. Create a folder named `exoplanet_eda_lab` and open it in VS Code.
2. Open the VS Code terminal (`Ctrl + ~`) and create a virtual environment: `python -m venv venv`
3. Activate the environment: `.\venv\Scripts\Activate.ps1`
4. Create `requirements.txt` (listing `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `ipykernel`) and run: `pip install -r requirements.txt`
5. Create `.gitignore` (listing `venv/`, `.venv/`, `__pycache__/`, `.ipynb_checkpoints/`, `*.csv`, and `*.db`).
6. Create `exoplanet_eda.ipynb`, `README.md`, and `LAB_NOTES.md`.
7. Initialize Git, commit, link your GitHub repository, and push to `origin main`.

---

### Pillar 1: Ingestion & Capture

#### Cell 1 — Download & Cache Raw Exoplanet Data

1. **Header Above Cell 1:** Add a Markdown cell: `## [Pillar 1: Ingestion & Capture] — Cell 1: Download & Cache Raw Exoplanet Data`
2. **Imports & Variables:** Import `os` and `pandas as pd`. Create two string variables: `RAW_FILE` (`"exoplanets_raw.csv"`) and `TAP_URL` (the NASA query link).
3. **Offline Cache Check (`if / else`):**
   - If `os.path.exists(RAW_FILE)` is true, load `df_raw = pd.read_csv(RAW_FILE)`.
   - Otherwise, download `df_raw = pd.read_csv(TAP_URL)` from NASA and save it locally with `df_raw.to_csv(RAW_FILE, index=False)`.
4. **Verify Table:** Print `df_raw.shape` (`6470 rows, 12 columns`) and display `df_raw.head()`.
5. **Reference Below Cell 1:** Add a Markdown cell below the output containing the 12-column dictionary.
6. **Git Checkpoint:** Commit with `feat(ingest): add NASA exoplanet download with local CSV caching`

---

### Pillar 2: Preparation & Cleaning

#### Cell 2 — Audit Data Types & Count Missing Values

1. **Header Above Cell 2:** Add a Markdown cell: `## [Pillar 2: Preparation & Cleaning] — Cell 2: Audit Data Types & Count Missing Values`
2. **Check Schema:** Run `df_raw.info()` to inspect column data types (`str`, `int64`, `float64`) and non-empty counts.
3. **Count Missing Data:** Calculate `missing_count = df_raw.isna().sum()` and `missing_pct = (df_raw.isna().mean() * 100).round(2)`.
4. **Display Summary Table:** Combine both into `missing_summary = pd.DataFrame({"Missing Count": missing_count, "Missing %": missing_pct})` and place `missing_summary` on the final line.
5. **Git Checkpoint:** Commit with `feat(clean): add schema info and missing value summary table in cell 2`

#### Cell 3 — Audit Number Ranges, Skew & Category Counts

1. **Header Above Cell 3:** Add a Markdown cell: `## [Pillar 2: Preparation & Cleaning] — Cell 3: Audit Number Ranges, Skew & Category Counts`
2. **Count Categories:** Run `print(df_raw["discoverymethod"].value_counts())` to see how planets were found.
3. **Inspect Number Ranges:** Run `df_raw.describe().round(2)` on the final line to check `min`, `50%` (median), `mean`, `std`, and `max`.
4. **Reference Below Cell 3:** Add a Markdown cell below the output explaining the Discovery Methods and `.describe()` rows.
5. **Git Checkpoint:** Commit with `feat(clean): add summary stats and discovery method audit in cell 3`

#### Cell 4 — Filter Incomplete Rows & Save Clean Dataset

1. **Header Above Cell 4:** Add a Markdown cell: `## [Pillar 2: Preparation & Cleaning] — Cell 4: Filter Incomplete Rows & Save Clean Dataset`
2. **Drop Incomplete Rows:** Create `df_clean` using `df_raw.dropna(subset=["pl_orbper", "pl_rade", "pl_bmasse", "st_teff"]).copy()`.
3. **Print Row Audit:** Print the row count before (`6470`), after (`6019`), and how many rows were removed (`451`).
4. **Save Clean File:** Save to disk with `df_clean.to_csv("exoplanets_clean.csv", index=False)`.
5. **Git Checkpoint:** Commit with `feat(clean): drop incomplete physical rows and save clean dataset in cell 4`

---

### Pillar 3: Exploration & Analysis

#### Cell 5 — Group by Discovery Method to Expose Telescope Biases

1. **Header Above Cell 5:** Add a Markdown cell: `## [Pillar 3: Exploration & Analysis] — Cell 5: Group by Discovery Method to Expose Telescope Biases`
2. **Group with Named Aggregation:** Group `df_clean` by `"discoverymethod"` and pass keyword pairs directly inside `.agg(...)`:
   - `planet_count=("pl_name", "count")`
   - `median_period_days=("pl_orbper", "median")`
   - `median_radius_earth=("pl_rade", "median")`
   - `median_mass_earth=("pl_bmasse", "median")`
   - `median_dist_pc=("sy_dist", "median")`
3. **Round and Sort:** Chain `.round(2).sort_values(by="planet_count", ascending=False)` onto the end and display `method_summary`.
4. **Reference Below Cell 5:** Add a Markdown cell below the output defining the summary columns and discovery methods.
5. **Git Checkpoint:** Commit with `feat(eda): add discovery method groupby median comparison in cell 5`

#### Cell 6 — Feature Engineering: Calculate Planet Density & Size Classes

1. **Header Above Cell 6:** Add a Markdown cell: `## [Pillar 3: Exploration & Analysis] — Cell 6: Engineer Bulk Density & Planet Size Classes`
2. **Calculate Planet Bulk Density (`density_gcc`):**
   - `df_clean["density_gcc"] = (df_clean["pl_bmasse"] / (df_clean["pl_rade"] ** 3)) * 5.51`
3. **Create Planet Size Categories (`size_class`) Using `pd.cut()`:**
   - Pass `df_clean["pl_rade"]` as the first argument inside `pd.cut(...)` with `bins=[0, 1.25, 2.0, 4.0, 100]` and `labels=["1. Earth-sized (<1.25 R_E)", "2. Super-Earth (1.25-2.0 R_E)", "3. Sub-Neptune (2.0-4.0 R_E)", "4. Gas Giant (>4.0 R_E)"]`.
4. **Group by `size_class`:**
   - Use `df_clean.groupby("size_class", observed=True).agg(...)` to compute `planet_count`, `median_radius`, `median_mass`, `median_density_gcc`, and `median_orbper_days`, chain `.round(2)`, and display `size_summary`.
5. **Reference Below Cell 6:** Add a Markdown cell below the output explaining how density drops across the `2.0 R_E` boundary.
6. **Git Checkpoint:** Commit with `feat(eda): engineer bulk density and size class comparison in cell 6`

---

### Pillar 4: Communication & Action

#### Cell 7 — Visualize the Radius Distribution & Log-Log Mass-Radius Relationship

1. **Header Above Cell 7:** Add a Markdown cell: `## [Pillar 4: Communication & Action] — Cell 7: Visualize Planet Radius & Mass-Radius Distributions`
2. **Import Plotting Libraries:** Import `matplotlib.pyplot as plt` and `seaborn as sns`.
3. **Create a 2-Panel Side-by-Side Figure:** Run `fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))`.
4. **Left Chart (`ax1`) — Histogram of Small Planet Radii (`< 5.0 R_E`):**
   - Filter `small_planets = df_clean[df_clean["pl_rade"] < 5.0]`.
   - Plot `sns.histplot(data=small_planets, x="pl_rade", bins=40, kde=True, ax=ax1)` and set title/labels on `ax1`.
5. **Right Chart (`ax2`) — Mass vs. Radius Scatter Plot on Log-Log Axes:**
   - Plot `sns.scatterplot(data=df_clean, x="pl_bmasse", y="pl_rade", hue="size_class", alpha=0.6, ax=ax2)`.
   - Set `ax2.set_xscale("log")` and `ax2.set_yscale("log")`, and set title/labels on `ax2`.
6. **Format, Save PNG, and Display:** Run `plt.tight_layout()`, `plt.savefig("exoplanet_distributions.png", dpi=300)`, and `plt.show()`.
7. **Reference Below Cell 7:** Add a Markdown cell below the charts explaining the Fulton Radius Valley (`~1.8-2.0 R_E` dip) and Gas Giant gravitational compression.
8. **Git Checkpoint:** Commit with `feat(viz): add radius histogram and log-log mass-radius scatter plot in cell 7`

#### Cell 8 — Load Cleaned Table into SQLite & Run SQL Queries

1. **Header Above Cell 8:** Add a Markdown cell: `## [Pillar 4: Communication & Action] — Cell 8: Load Cleaned Data into SQLite & Query Nearby Rocky Worlds`
2. **Import `sqlite3` and Connect:**
   - Import `sqlite3` and open a connection to a local database file: `conn = sqlite3.connect("exoplanets.db")`.
3. **Write `df_clean` to a Relational SQL Table:**
   - Load all `6,019` rows into a table named `"exoplanets"`: `df_clean.to_sql("exoplanets", conn, if_exists="replace", index=False)`.
4. **Execute an Analytical SQL Query (`pd.read_sql_query`):**
   - Define a multi-line SQL string `query = """ ... """` that selects the top 10 closest rocky planets (`pl_rade <= 2.0`, `density_gcc >= 4.0`, `sy_dist <= 30.0`) ordered by distance (`ORDER BY sy_dist ASC LIMIT 10`).
   - Run `nearby_rocky_df = pd.read_sql_query(query, conn)`.
5. **Close Connection and Display Result:**
   - Call `conn.close()` to release the database file cleanly, and put `nearby_rocky_df` on the final line of Cell 8.
6. **Reference Below Cell 8:** Add a Markdown cell below the output explaining the `Proxima Centauri` and `Barnard's Star` rocky planet findings.
7. **Git Checkpoint:** Commit with `feat(db): load cleaned exoplanets into SQLite and query nearby rocky worlds in cell 8`

---

---

## PART II: Concepts, Rationale & Q&A Guide

### Workspace & Environment Concepts

| Question / Topic                                                  | Plain-English Explanation                                                                                                                                                                                                    |
| :---------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Why start with Exoplanets before Neo-Assyrian texts?**          | The Exoplanet catalog is already a clean table of rows and columns, letting you learn core `pandas` commands first. Cuneiform text requires complex string/regex cleanup before you even have a table.                       |
| **Why use Native Windows instead of an Ubuntu VM?**               | On an 8GB RAM machine, a graphical VM splits memory in half (~4GB each), causing lag. Native Windows gives VS Code and Jupyter full access to all 8GB of RAM.                                                                |
| **Why use VS Code's built-in PowerShell terminal?**               | It automatically opens inside your project folder and activates your `(venv)` environment whenever you open a new terminal tab.                                                                                              |
| **Why does `python -m venv venv` say `venv` twice?**              | `-m venv` runs Python's built-in environment tool; the second `venv` is the name of the folder it creates on your computer.                                                                                                  |
| **Why list only 6 packages in `requirements.txt`?**               | Listing only direct tools (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `ipykernel`) keeps the file clean instead of dumping 40+ hidden sub-packages from `pip freeze`.                                              |
| **Why does `git branch -M main` come after the first commit?**    | Git doesn't create a real branch on disk until the first commit is saved. Running `git branch -M main` right after ensures the branch is named `main`.                                                                       |
| **What does the Git `LF will be replaced by CRLF` warning mean?** | Linux uses one invisible character (`LF`) for line breaks; Windows uses two (`CRLF`). Git is simply matching Windows formatting automatically. It is completely harmless.                                                    |
| **Where should Markdown cells go in a Jupyter Notebook?**         | **Sandwich Pattern:** Put a 1-line Pillar/Step header **above** the code cell, and put glossaries, definitions, and takeaways **below** the code cell so they sit right next to the output table.                            |
| **Why did my 1-line Markdown header look tiny at first?**         | Markdown controls text size by starting the line with `#` symbols followed by a space (`# ` = Largest Title, `## ` = Large Section Header, `### ` = Medium Sub-header). Without `# `, Markdown prints tiny normal body text. |
| **Why did `LAB_NOTES.md` look messy in VS Code at first?**        | Viewing raw `.md` source code shows all the `#`, `**`, and `                                                                                                                                                                 | ` formatting symbols. Pressing **`Ctrl + Shift + V`\*\* in VS Code opens the clean, formatted Reader Preview. |

---

### Pillar 1: Ingestion & Capture (Cell 1 Rationale)

- **Why We Picked `pscomppars` Over `ps`:** NASA's general `ps` table has multiple rows for the same planet (one per published paper), which would count famous planets 15 times and ruin our averages. The `pscomppars` table has **strictly 1 row per planet** (`6,470` unique planets).
- **Why We Cache `exoplanets_raw.csv` Locally:** Checking `os.path.exists()` means we only download from NASA once. Every future run loads from your hard drive in 0.1 seconds, works offline, and locks in our exact `6,470` planet count so new weekly NASA discoveries don't change our numbers mid-project.
- **Why We Use `index=False` in `.to_csv()`:** Stops `pandas` from saving its automatic row numbers (`0, 1, 2...`) as an extra unnamed column inside the CSV file.
- **Why We Use f-strings (`print(f"... {df_raw.shape}")`):** Python won't let you combine text and tuples with `+`. Putting `f` before the quote lets you drop any variable straight into `{...}`.
- **Why `df_raw.head()` vs `print(df_raw.head())` Matters:** Leaving `df_raw.head()` alone on the last line renders a shaded HTML webpage table in VS Code. Wrapping it in `print()` outputs plain text that copies and pastes cleanly.

---

### Pillar 2: Preparation & Cleaning (Cells 2, 3 & 4 Rationale)

#### Why Are Cells 2 and 3 Part of "Preparation & Cleaning"?

Before you can clean a dataset (Cell 4), you have to **diagnose what is broken, missing, or skewed** (Cells 2 and 3). Looking only at the first 5 rows in Cell 1 (`df_raw.head()`) tricked us into thinking there were zero missing values because the top 5 rows happened to be complete!

#### Cell 2 Takeaways — What Is Missing Out of 6,470 Planets?

- **How `.isna().sum()` Works:** `.isna()` turns every cell into `True` (`1` if missing) or `False` (`0` if present). `.sum()` adds the `1`s to count missing rows; `.mean() * 100` calculates the percentage missing.
- **Why Combine into a Mini DataFrame (`missing_summary`):** Putting counts and percentages side-by-side in `pd.DataFrame({...})` prevents VS Code from cutting off the output at its 30-line plain-text limit.
- **Our Missing Data Findings:**
  1. **`pl_eqt` (Planet Temperature): 540 missing (8.35%).** Requires knowing both star brightness and orbital distance; if either is missing, temperature can't be calculated.
  2. **`pl_orbper` (Orbital Period): 356 missing (5.50%).** Distant planets found by Imaging or one-time Microlensing events haven't been watched long enough to complete a full orbit.
  3. **`st_teff` (Star Temperature): 309 missing (4.78%).** Faint microlensing stars or dead pulsar remnants lack standard temperature readings.
  4. **`pl_rade` (50 missing), `pl_bmasse` (31 missing), `sy_dist` (28 missing):** Over 99% complete.

#### Cell 3 Takeaways — Category Dominance & Extreme Skew

- **Discovery Method Dominance (`.value_counts()`):**
  - **Transit (`4,799` planets / 74.2%)** and **Radial Velocity (`1,205` planets / 18.6%)** account for **92.8%** of all known exoplanets.
- **No Fake Error Codes (`min` row):** Some databases use `-999` for missing numbers. Checking `min` confirmed all minimums are real positive physics numbers (closest star `sy_dist = 1.30 pc` is Proxima Centauri!).
- **Why `Mean` vs. `50%` (Median) Is Huge Here:**
  - **`std` (Standard Deviation)** measures how spread out numbers are from the average.
  - In **Orbital Period (`pl_orbper`)**, the **`50%` median** planet orbits in **`10.64` days**, but the **`mean`** is **`70,021` days** (`7.00e+04`) because one extreme planet takes `402,000,000` days (`4.02e+08`), dragging the average up 6,500x!
  - In **Planet Mass (`pl_bmasse`)**, the **`50%` median** is **`9.32` Earths**, while the **`mean`** is **`404.41` Earths** due to massive gas giants (`max = 12,651.50` Earths).
  - **Rule of Thumb:** Whenever a column has giant outliers, always use the **`50%` Median** instead of the `Mean` to describe a typical row!

#### Cell 4 Takeaways — Results of Our Cleaning Filter (`6,470` $\rightarrow$ `6,019` Planets)

- **Why Drop Rows (`.dropna()`) Instead of Filling Averages (`.fillna()`):** Filling a missing planet's mass with the average mass (`404 Earths`) would invent fake physics. Instead, we filtered down to rows with real measurements for `["pl_orbper", "pl_rade", "pl_bmasse", "st_teff"]`.
- **Why Only 451 Rows Were Dropped (Instead of 746):** Even though `356 + 309 + 50 + 31 = 746` missing cells existed across those 4 columns, many missing values overlapped on the exact same rows (e.g., Microlensing planets missing both orbital period and star temperature). We retained **`6,019` complete planets (93.0% of the catalog)**.
- **Why Put `.copy()` at the End:** Creates `df_clean` as an independent table in memory so adding new engineered columns in Pillar 3 won't trigger `pandas` warnings.

---

### Pillar 3: Exploration & Analysis (Cells 5 & 6 Rationale)

#### Q15: Why did `.agg(["count", "median"])` repeat the `count` column 4 times, and how do we fix it?

- **Why It Happened:** Passing a list of functions (`["count", "median"]`) across 4 selected columns tells `pandas` to calculate both functions for every column (`4 × 2 = 8` columns total), cluttering the table with 4 identical count columns and making the medians harder to compare side-by-side.
- **The Fix (Named Aggregation):** By passing keyword pairs directly inside `.agg(planet_count=("pl_name", "count"), median_period_days=("pl_orbper", "median"), ...)`, we tell `pandas` to calculate the row count **only once** and line up all 4 median columns cleanly next to each other.

#### Q16: Why did defining `planet_count = ("pl_name", "count")` outside `.agg()` and passing `"planet_count"` in quotes cause an error?

- **The Reason:** Putting quotes around `"planet_count"` inside `.agg("planet_count")` tells `pandas` to look for a built-in math function named `"planet_count"` (like `"mean"` or `"sum"`), which doesn't exist. Named aggregations must be written as **keyword arguments (`new_col_name=("old_col", "func")`) directly inside the parentheses of `.agg(...)`**, just like `index=False` sits directly inside `.to_csv(...)`.

#### Q17: Why did `df_clean["pl_rade"].cut(...)` fail with an `AttributeError`, while `pd.cut(df_clean["pl_rade"], ...)` worked?

- **The Reason:** In `pandas`, some tools are **column methods** attached to a Series (like `.isna()`, `.sum()`, `.round()`), while others are **top-level library functions** attached to `pd.` (like `pd.read_csv()`, `pd.DataFrame()`, and `pd.cut()`). For `pd.cut()`, you call the function from `pd.` and pass the column `df_clean["pl_rade"]` as the first item inside the parentheses.

#### Cell 5 Takeaways — What the Grouped Medians Reveal About Telescope Bias

1. **Why Microlensing (`294` $\rightarrow$ `2`) and Imaging (`98` $\rightarrow$ `22`) Lost Most of Their Rows in Cell 4:**
   - Almost all `Microlensing` and `Imaging` planets were the exact rows missing `pl_orbper` or `st_teff` that we filtered out during cleaning!
2. **Transit vs. Radial Velocity (40x Difference in Orbit, 45x Difference in Mass):**
   - **Transit (`4,772` clean planets):** Median orbital period is **`8.01` days**, median radius is **`2.45` Earths**, median mass is **`6.98` Earths**, and median distance is **`455.48` parsecs**. Transit excels at finding small, close-in sub-Neptunes and super-Earths deep across the galaxy because close-in planets cross in front of their star frequently.
   - **Radial Velocity (`1,166` clean planets):** Median orbital period is **`319.70` days** (~1 Earth year), median radius is **`12.60` Earths**, median mass is **`314.65` Earths** (almost identical to Jupiter at `317.8` Earths!), and median distance is **`45.88` parsecs** (10x closer to Earth). Watching a star wobble via Doppler shift requires heavy gas giants orbiting bright, nearby stars.
3. **Direct Imaging (`22` clean planets — Extreme Outer Giants):**
   - Median orbital period is **`33,100` days (~90.6 Earth years)** and median mass is **`3,089.15` Earths** (~10x Jupiter!) around very close stars (**`41.24` parsecs**). Direct cameras can only resolve giant, glowing planets orbiting far outside their host star's glare.

#### Cell 6 Takeaways — What Our Bulk Density (`density_gcc`) Feature Engineering Proved

- **1. Earth-sized (`<1.25 R_E` — `597` planets):** Median radius `1.06 R_E`, median mass `1.32 M_E`, and **median density `5.67 g/cm³`** (virtually identical to Earth's `5.51 g/cm³` solid rock/iron composition).
- **2. Super-Earth (`1.25-2.0 R_E` — `1,203` planets):** Median radius `1.60 R_E`, median mass `3.33 M_E`, and **median density `4.43 g/cm³`** (predominantly rocky cores).
- **3. Sub-Neptune (`2.0-4.0 R_E` — `2,075` planets):** Median radius `2.63 R_E`, median mass `7.65 M_E`, and **median density plunges to `2.31 g/cm³`!** Once a planet crosses `~2.0` Earth radii, it holds onto a thick hydrogen/helium gas envelope that inflates its volume and cuts its density in half.
- **4. Gas Giant (`>4.0 R_E` — `2,144` planets):** Median radius `12.60 R_E`, median mass `292.40 M_E`, and **median density drops to `0.97 g/cm³`** (less dense than liquid water at `1.00 g/cm³`!).

---

### Pillar 4: Communication & Action (Cells 7 & 8 Rationale)

#### Q18: Why did `figsize=14, 5` and `df_clean[pl_rade]` cause syntax errors in Cell 7?

- **`figsize=(14, 5)`:** Because `plt.subplots()` takes multiple arguments separated by commas, width and height must be bundled together inside parentheses `(14, 5)` as a single two-number tuple.
- **`df_clean["pl_rade"]`:** Column names inside square brackets must always be strings wrapped in quotation marks (`"pl_rade"`), otherwise Python looks for an undefined variable named `pl_rade`.

#### Q19: Why did `pl.read_sql_query` fail in Cell 8 while `pd.read_sql_query` worked?

- **The Reason:** `read_sql_query()` is a top-level function inside the `pandas` library, which we imported under the alias **`pd`** in Cell 1 (`import pandas as pd`). Calling `pd.read_sql_query(query, conn)` runs the SQL query against the SQLite connection and returns the result as a formatted `pandas` DataFrame.

#### Cell 7 Visual Takeaways — The Fulton Radius Valley & Gas Giant Compression

1. **Why Is There a Dip Around `1.8 - 2.0 R_E` in the Left Histogram? (The Fulton Radius Valley):**
   - That dip between `~1.4 R_E` (Super-Earths) and `~2.4 R_E` (Sub-Neptunes) is a real physical phenomenon caused by **photoevaporation**. Close-in planets get blasted by radiation from their host star: smaller rocky cores lose their gas atmosphere completely and shrink down to the `1.4 R_E` peak, while heavier cores hold onto their hydrogen/helium envelopes and stay inflated at `2.4 R_E`.
2. **Why Do Gas Giants Level Off Horizontally Around `~11–13 R_E` in the Right Scatter Plot?**
   - Above `~100` Earth masses (Saturn/Jupiter mass) all the way up to `10,000+` Earth masses (the Brown Dwarf threshold), strong gravity compresses the interior hydrogen gas into **electron-degenerate matter**. Adding more mass packs matter tighter into the core without making the planet physically wider—so gas giants stay roughly the size of Jupiter (`~11–13 R_E`) while becoming much denser!

#### Cell 8 Takeaways — Bridging Pandas to SQLite & Our Closest Rocky Neighbors

- **Why Load Cleaned Data into SQLite (`exoplanets.db`):** In production data pipelines, cleaned DataFrames are loaded into a relational database table (`df_clean.to_sql(...)`) so downstream analysts and applications can query the data using standard SQL (`SELECT ... FROM ... WHERE ... ORDER BY ...`).
- **Top 10 Closest Rocky Worlds (`sy_dist <= 30 pc`, `pl_rade <= 2.0 R_E`, `density_gcc >= 4.0 g/cm³`):**
  - **Slots 0 & 1 (`Proxima Cen d` & `Proxima Cen b` at `1.30 pc` / `4.24 ly`):** Orbiting the closest star to our Sun, `Proxima Cen b` is an Earth-sized rocky world (`1.02 R_E`, `1.055 M_E`, `5.48 g/cm³`) on an `11.18`-day habitable-zone orbit.
  - **Slots 2–5 (`Barnard b, c, d, e` at `1.83 pc` / `5.96 ly`):** Four compact sub-Earth rocky planets (`0.64–0.74 R_E`) orbiting the second-closest star system to Earth.
  - **Why All 10 Were Found by `Radial Velocity`:** Because `Transit` requires a lucky edge-on orbital alignment with Earth, almost none of our immediate stellar neighbors happen to transit. Instead, all 10 closest rocky exoplanets were discovered by measuring the Doppler gravitational wobble (`Radial Velocity`) of nearby red dwarf stars.
