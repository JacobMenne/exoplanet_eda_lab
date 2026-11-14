# NASA Exoplanet Archive: Exploratory Data Science Lab

---

## PART I: Step-by-Step Build Manual

### Step 1.0: Repository & Environment Setup

1. Create project directory `exoplanet_eda_lab` and navigate into it.
2. Initialize a local Git repository on branch `main`.
3. Create and activate a Python virtual environment; install `pandas`, `numpy`, `matplotlib`, `seaborn`, and `scipy`, then freeze dependencies to `requirements.txt`.
4. Create `.gitignore` excluding virtual environments (`venv/`, `.venv/`), bytecode (`__pycache__/`), OS artifacts (`.DS_Store`), and notebook checkpoints (`.ipynb_checkpoints/`).
5. Create empty project files: `exoplanet_eda.ipynb`, `README.md`, and `LAB_NOTES.md`.
6. Stage, commit (`chore: initialize repository structure, gitignore, and lab notes`), link remote GitHub repository, and push to `origin main`.

### Step 1.1: Data Ingestion & Offline Caching (Cell 1)

1. In Cell 1 of `exoplanet_eda.ipynb`, import `os` and `pandas as pd`.
2. Define the local cache filename (`exoplanets_raw.csv`) and the remote NASA TAP query URL targeting 12 specific columns from the `pscomppars` table:
   `https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query=select+pl_name,hostname,discoverymethod,disc_year,sy_snum,sy_pnum,pl_orbper,pl_rade,pl_bmasse,pl_eqt,st_teff,sy_dist+from+pscomppars&format=csv`
3. Implement an `os.path.exists()` check:
   - **If local file exists:** Load `exoplanets_raw.csv` into `df_raw` via `pd.read_csv()`.
   - **If local file does not exist:** Query the NASA URL via `pd.read_csv()`, assign to `df_raw`, and save to `exoplanets_raw.csv` with `index=False`.
4. Print the dataset dimensions (`df_raw.shape`) and display the first 5 rows (`df_raw.head()`).
5. Stage changes and commit (`feat(ingest): add NASA pscomppars TAP ingestion with local CSV caching`).

---

## PART II: Data Science Concepts & Architectural Rationale

### Step 1.0: Version Control & Environment Isolation

- **What This Step Does:** Establishes a reproducible workspace with isolated Python dependencies and a clean Git commit history before any data is downloaded or mutated.
- **Why Each Decision Was Made:**
  - _Environment Isolation (`venv` + `requirements.txt`):_ Data science libraries like `pandas` and `numpy` frequently change default behaviors across major versions (e.g., how missing values or string types are handled). Pinning dependencies ensures the analysis runs identically across machines.
  - _Git Hygiene (`.gitignore`):_ Excluding `.ipynb_checkpoints/` and virtual environment binaries keeps Git diffs focused strictly on human-written code and documentation.
  - _Atomic Conventional Commits:_ Committing at the boundary of each analytical stage (`chore`, `feat(ingest)`, `feat(clean)`, `feat(eda)`, `feat(viz)`) creates a clear audit trail so any data transformation can be traced or rolled back independently.

### Step 1.1: Data Provenance, Table Granularity & Offline-First Ingestion

- **What This Step Does:** Pulls a 12-column tabular extract of confirmed exoplanets from the NASA Exoplanet Archive's Table Access Protocol (TAP) API and persists an immutable raw snapshot (`exoplanets_raw.csv`) to local disk.
- **Why Each Decision Was Made:**
  - _Granularity Selection (`pscomppars` vs. `ps`):_ In relational data science, defining the **unit of observation** (what a single row represents) is mandatory before computing statistics. NASA's `ps` table contains multiple rows per planet (one per published study), which violates the assumption of independent observations and biases means/counts toward famous planets. The `pscomppars` (Composite Parameters) table enforces **strictly one row per confirmed planet**, making it valid for population-level statistics.
  - _Server-Side Column Projection (12 vs. 300+ Columns):_ Rather than downloading all 300+ columns (error bounds, limit flags, literature references) and dropping them in memory, projecting only the 12 required columns inside the TAP URL query reduces network payload size by >90% and speeds up I/O parsing.
  - _Immutable Raw Caching (`exoplanets_raw.csv`):_ Live scientific databases update continuously as new planets are confirmed. Caching the initial pull guarantees **analytical reproducibility** (preventing row counts and summary statistics from shifting mid-project), eliminates external network latency/outages on subsequent runs, and preserves an untouched baseline if downstream cleaning logic needs to be reset.
  - _Suppressing Index Serialization (`index=False`):_ By default, `pandas.to_csv()` writes the DataFrame's integer row index as an unnamed first column (`Unnamed: 0`), which pollutes the schema when reloaded. Passing `index=False` keeps the stored CSV schema identical to the remote source.
