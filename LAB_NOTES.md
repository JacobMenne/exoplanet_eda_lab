# NASA Exoplanet Archive — Data Science Lab Notes

> **How to read this file in VS Code:** Press **`Ctrl + Shift + V`** to switch from raw text to the clean, formatted Reader View!

---

## PART I: Step-by-Step Build Manual

### Workspace & Git Setup (Before Coding)

1. Create a folder named `exoplanet_eda_lab` and open it in VS Code.
2. Open the VS Code terminal (`Ctrl + ~`) and create a virtual environment: `python -m venv venv`
3. Activate the environment: `.\venv\Scripts\Activate.ps1`
4. Create `requirements.txt` (listing `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `ipykernel`) and run: `pip install -r requirements.txt`
5. Create `.gitignore` (listing `venv/`, `.venv/`, `__pycache__/`, `.ipynb_checkpoints/`, and `*.csv`).
6. Create `exoplanet_eda.ipynb`, `README.md`, and `LAB_NOTES.md`.
7. Initialize Git, commit, link your GitHub repository, and push to `origin main`.

---

### Pillar 1: Ingestion & Capture

#### Cell 1 — Download & Cache Raw Exoplanet Data

1. **Header Above Cell 1:** Add a Markdown cell: `### [Pillar 1: Ingestion & Capture] — Cell 1: Download & Cache Raw Exoplanet Data`
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

1. **Header Above Cell 2:** Add a Markdown cell: `### [Pillar 2: Preparation & Cleaning] — Cell 2: Audit Data Types & Count Missing Values`
2. **Check Schema:** Run `df_raw.info()` to inspect column data types (`str`, `int64`, `float64`) and non-empty counts.
3. **Count Missing Data:** Calculate `missing_count = df_raw.isna().sum()` and `missing_pct = (df_raw.isna().mean() * 100).round(2)`.
4. **Display Summary Table:** Combine both into `missing_summary = pd.DataFrame({"Missing Count": missing_count, "Missing %": missing_pct})` and place `missing_summary` on the final line.
5. **Git Checkpoint:** Commit with `feat(clean): add schema info and missing value summary table in cell 2`

#### Cell 3 — Audit Number Ranges, Skew & Category Counts

1. **Header Above Cell 3:** Add a Markdown cell: `### [Pillar 2: Preparation & Cleaning] — Cell 3: Audit Number Ranges, Skew & Category Counts`
2. **Count Categories:** Run `print(df_raw["discoverymethod"].value_counts())` to see how planets were found.
3. **Inspect Number Ranges:** Run `df_raw.describe().round(2)` on the final line to check `min`, `50%` (median), `mean`, `std`, and `max`.
4. **Reference Below Cell 3:** Add a Markdown cell below the output explaining the Discovery Methods and `.describe()` rows.
5. **Git Checkpoint:** Commit with `feat(clean): add summary stats and discovery method audit in cell 3`

#### Cell 4 — Filter Incomplete Rows & Save Clean Dataset

1. **Header Above Cell 4:** Add a Markdown cell: `### [Pillar 2: Preparation & Cleaning] — Cell 4: Filter Incomplete Rows & Save Clean Dataset`
2. **Drop Incomplete Rows:** Create `df_clean` using `df_raw.dropna(subset=["pl_orbper", "pl_rade", "pl_bmasse", "st_teff"]).copy()`.
3. **Print Row Audit:** Print the row count before (`6470`), after (`6019`), and how many rows were removed (`451`).
4. **Save Clean File:** Save to disk with `df_clean.to_csv("exoplanets_clean.csv", index=False)`.
5. **Git Checkpoint:** Commit with `feat(clean): drop incomplete physical rows and save clean dataset in cell 4`

---

### Pillar 3: Exploration & Analysis

#### Cell 5 — Group by Discovery Method to Expose Telescope Biases

1. **Header Above Cell 5:** Add a Markdown cell: `### [Pillar 3: Exploration & Analysis] — Cell 5: Group by Discovery Method to Expose Telescope Biases`
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

1. **Header Above Cell 6:** Add a Markdown cell: `### [Pillar 3: Exploration & Analysis] — Cell 6: Engineer Bulk Density & Planet Size Classes`
2. **Calculate Planet Bulk Density (`density_gcc`):**
   - Create a new column `"density_gcc"` (in grams per cubic centimeter, where Earth = `5.51`) by dividing `df_clean["pl_bmasse"]` by `df_clean["pl_rade"] ** 3` and multiplying by `5.51`:
     `df_clean["density_gcc"] = (df_clean["pl_bmasse"] / (df_clean["pl_rade"] ** 3)) * 5.51`
3. **Create Planet Size Categories (`size_class`) Using `pd.cut()`:**
   - Slice `df_clean["pl_rade"]` into 4 astronomical size bins using `pd.cut()`:
     - `bins=[0, 1.25, 2.0, 4.0, 100]`
     - `labels=["1. Earth-sized (<1.25 R_E)", "2. Super-Earth (1.25-2.0 R_E)", "3. Sub-Neptune (2.0-4.0 R_E)", "4. Gas Giant (>4.0 R_E)"]`
4. **Group by `size_class` to Compare Rocky vs. Gas Worlds:**
   - Group `df_clean` by `"size_class"` (passing `observed=False` or `observed=True` inside `.groupby("size_class", observed=True)`) and use `.agg(...)` to compute:
     - `planet_count=("pl_name", "count")`
     - `median_radius=("pl_rade", "median")`
     - `median_mass=("pl_bmasse", "median")`
     - `median_density_gcc=("density_gcc", "median")`
     - `median_orbper_days=("pl_orbper", "median")`
   - Chain `.round(2)` and display the resulting `size_summary` table on the final line.
5. **Git Checkpoint:** Commit with `feat(eda): engineer bulk density and size class comparison in cell 6`

---

---

## PART II: Concepts, Rationale & Q&A Guide

### Workspace & Environment Concepts

| Question / Topic                                                  | Plain-English Explanation                                                                                                                                                                              |
| :---------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Why start with Exoplanets before Neo-Assyrian texts?**          | The Exoplanet catalog is already a clean table of rows and columns, letting you learn core `pandas` commands first. Cuneiform text requires complex string/regex cleanup before you even have a table. |
| **Why use Native Windows instead of an Ubuntu VM?**               | On an 8GB RAM machine, a graphical VM splits memory in half (~4GB each), causing lag. Native Windows gives VS Code and Jupyter full access to all 8GB of RAM.                                          |
| **Why use VS Code's built-in PowerShell terminal?**               | It automatically opens inside your project folder and activates your `(venv)` environment whenever you open a new terminal tab.                                                                        |
| **Why does `python -m venv venv` say `venv` twice?**              | `-m venv` runs Python's built-in environment tool; the second `venv` is the name of the folder it creates on your computer.                                                                            |
| **Why list only 6 packages in `requirements.txt`?**               | Listing only direct tools (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `ipykernel`) keeps the file clean instead of dumping 40+ hidden sub-packages from `pip freeze`.                        |
| **Why does `git branch -M main` come after the first commit?**    | Git doesn't create a real branch on disk until the first commit is saved. Running `git branch -M main` right after ensures the branch is named `main`.                                                 |
| **What does the Git `LF will be replaced by CRLF` warning mean?** | Linux uses one invisible character (`LF`) for line breaks; Windows uses two (`CRLF`). Git is simply matching Windows formatting automatically. It is completely harmless.                              |
| **Where should Markdown cells go in a Jupyter Notebook?**         | **Sandwich Pattern:** Put a 1-line Pillar/Step header **above** the code cell, and put glossaries, definitions, and takeaways **below** the code cell so they sit right next to the output table.      |
| **Why did `LAB_NOTES.md` look messy in VS Code at first?**        | Viewing raw `.md` source code shows all the `#`, `**`, and `                                                                                                                                           | ` formatting symbols. Pressing **`Ctrl + Shift + V`\*\* in VS Code opens the clean, formatted Reader Preview. |

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

- **The Reason:** Putting quotes around `"planet_count"` inside `.agg("planet_count")` tells `pandas` to look for a built-in math function named `"planet_count"` (like `"mean"` or `"sum"`), which doesn't exist. Named aggregations must be written as **keyword arguments (`new_col_name=("old_col", "func")`) directly inside the parentheses of `.agg(...)`**, just like `index=False` sits directly inside `.to_csv(...)`. Wrapping the whole chain in outer parentheses `(...)` lets you format each step on its own clean, indented line.

#### Cell 5 Takeaways — What the Grouped Medians Reveal About Telescope Bias

1. **Why Microlensing (`294` $\rightarrow$ `2`) and Imaging (`98` $\rightarrow$ `22`) Lost Most of Their Rows in Cell 4:**
   - Almost all `Microlensing` and `Imaging` planets were the exact rows missing `pl_orbper` or `st_teff` that we filtered out during cleaning!
2. **Transit vs. Radial Velocity (40x Difference in Orbit, 45x Difference in Mass):**
   - **Transit (`4,772` clean planets):** Median orbital period is **`8.01` days**, median radius is **`2.45` Earths**, median mass is **`6.98` Earths**, and median distance is **`455.48` parsecs**. Transit excels at finding small, close-in sub-Neptunes and super-Earths deep across the galaxy because close-in planets cross in front of their star frequently.
   - **Radial Velocity (`1,166` clean planets):** Median orbital period is **`319.70` days** (~1 Earth year), median radius is **`12.60` Earths**, median mass is **`314.65` Earths** (almost identical to Jupiter at `317.8` Earths!), and median distance is **`45.88` parsecs** (10x closer to Earth). Watching a star wobble via Doppler shift requires heavy gas giants orbiting bright, nearby stars.
3. **Direct Imaging (`22` clean planets — Extreme Outer Giants):**
   - Median orbital period is **`33,100` days (~90.6 Earth years)** and median mass is **`3,089.15` Earths** (~10x Jupiter!) around very close stars (**`41.24` parsecs**). Direct cameras can only resolve giant, glowing planets orbiting far outside their host star's glare.

#### Cell 6 Concepts — What Is Feature Engineering (`density_gcc` & `size_class`)?

- **What Feature Engineering Means:** Creating brand-new, physically meaningful columns from raw columns to reveal patterns that aren't obvious from raw numbers alone.
- **The Bulk Density Formula (`density_gcc`):** Since a sphere's volume scales with $\text{Radius}^3$, dividing Earth-masses (`pl_bmasse`) by Earth-radii cubed (`pl_rade ** 3`) and multiplying by Earth's density (`5.51 g/cm³`) gives each planet's bulk density. Rocky planets have high densities (`~4 to 7 g/cm³`), whereas puffy gas-envelope planets have low densities (`~0.5 to 2.5 g/cm³`).
- **How `pd.cut()` Works:** Takes a continuous number column (`pl_rade`) and slices it into named category buckets based on boundary numbers (`bins=[0, 1.25, 2.0, 4.0, 100]`), letting us group planets by physical class (`Earth-sized`, `Super-Earth`, `Sub-Neptune`, `Gas Giant`).
