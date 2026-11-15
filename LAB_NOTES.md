# NASA Exoplanet Archive: Exploratory Data Science Lab

---

## PART I: Step-by-Step Build Manual (Plain-English Checklist)

### Step 1.0: Setting Up the Folder, Virtual Environment, and GitHub Repo

1. **Create the project folder:** Make a new folder on your computer named `exoplanet_eda_lab` and open it directly inside VS Code.
2. **Open the VS Code terminal:** Open the built-in terminal inside VS Code (`Ctrl + ~`). Using PowerShell inside VS Code is completely fine and automatically opens inside your project folder.
3. **Create a Python virtual environment (`venv`):**
   - Run `python -m venv venv` in the terminal. _(The first `venv` calls Python's built-in environment tool; the second `venv` is the name of the folder it creates)._
   - Activate the environment by running `.\venv\Scripts\Activate.ps1`. Confirm that `(venv)` appears in green at the far left of your terminal prompt.
   - _(Optional)_ If `pip` displays a notice that a newer version is available, run `python.exe -m pip install --upgrade pip`.
4. **Create `requirements.txt` and install packages:**
   - Create a file named `requirements.txt` and list the 6 core libraries on separate lines: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, and `ipykernel`.
   - Run `pip install -r requirements.txt` to install all 6 packages into your active `(venv)`.
5. **Create `.gitignore`:**
   - Create a file named `.gitignore` and add these lines so Git ignores heavy folders, temporary files, and downloaded CSV datasets:
     ```text
     venv/
     .venv/
     __pycache__/
     .ipynb_checkpoints/
     *.csv
     ```
6. **Create your project files:**
   - Create an empty notebook named `exoplanet_eda.ipynb` (and select your `venv` Python kernel in the top-right corner of VS Code).
   - Create `README.md` (with the Data Dictionary table) and `LAB_NOTES.md`.
7. **Initialize Git and push to GitHub:**
   - Create an empty repository named `exoplanet_eda_lab` on GitHub.com (do not check the box to add a README on GitHub).
   - Run these commands in your VS Code terminal:
     ```powershell
     git init
     git add .
     git commit -m "chore: initialize repository structure, gitignore, and lab notes"
     git branch -M main
     git remote add origin [https://github.com/JacobMenne/exoplanet_eda_lab.git](https://github.com/JacobMenne/exoplanet_eda_lab.git)
     git push -u origin main
     ```

### Step 1.1: Downloading and Saving the Exoplanet Data (Cell 1)

1. **Import your tools:** At the very top of Cell 1 in `exoplanet_eda.ipynb`, write two import lines: one to import `os` (which lets Python check files on your hard drive) and one to import `pandas as pd` (which handles data tables).
2. **Store the filename and web link in variables:** Create two standard Python string variables using `=` and quotation marks (`"..."`) so you don't have to paste long text strings inside your functions later:
   - `RAW_FILE = "exoplanets_raw.csv"`
   - `TAP_URL = "https://exoplanetarchive.ipac.caltech.edu/TAP/sync?query=select+pl_name,hostname,discoverymethod,disc_year,sy_snum,sy_pnum,pl_orbper,pl_rade,pl_bmasse,pl_eqt,st_teff,sy_dist+from+pscomppars&format=csv"`
3. **Check if the file is already on your computer (`if / else`):**
   - Write an `if` statement using `os.path.exists(RAW_FILE):`.
   - **Inside the `if` block (indented):** Load the file into `df_raw` using `pd.read_csv(RAW_FILE)` and print a message saying it loaded from disk.
   - **Inside the `else:` block (indented):** Download the table into `df_raw` using `pd.read_csv(TAP_URL)`, save it to disk using `df_raw.to_csv(RAW_FILE, index=False)`, and print a message saying it downloaded from the web.
4. **Inspect the loaded table (un-indented at the bottom of Cell 1):**
   - Use an f-string `print(f"Dataset dimensions (Rows, Columns): {df_raw.shape}")` to print the row and column counts (`6470` rows, `12` columns).
   - On the very last line of the cell, write `df_raw.head()` so Jupyter displays a preview table of the first 5 planets.
5. **Save and commit to Git:**
   - Save your notebook (`Ctrl + S`), then stage, commit (`feat(ingest): add NASA exoplanet download with local CSV caching`), and push to GitHub.

### Step 2.1: Checking Data Types and Counting Missing Values (Cell 2)

1. **Check column data types and row counts:** In Cell 2, run `df_raw.info()` to see the data type (`str` for text, `int64` for whole numbers, `float64` for decimals) and non-empty row counts for all 12 columns.
2. **Count exact missing values (`NaN`) per column:** Store the count of missing rows per column in a variable: `missing_count = df_raw.isna().sum()`.
3. **Calculate the percentage of missing data per column:** Store the missing percentage rounded to 2 decimal places in a second variable: `missing_pct = (df_raw.isna().mean() * 100).round(2)`.
4. **Combine into a single side-by-side summary table:** Pass a Python dictionary (`{"Missing Count": missing_count, "Missing %": missing_pct}`) into `pd.DataFrame(...)` to combine both lists side-by-side into `missing_summary`, and put `missing_summary` on the final line of the cell so Jupyter renders a compact HTML table without hitting VS Code's 30-line text limit.
5. **Save and commit to Git:** Save the notebook and commit (`feat(eda): add schema info and missing value summary table in cell 2`).

### Step 2.2: Checking Summary Math, Ranges, and Weird Outliers (Cell 3)

1. **Count planets by discovery method:** At the top of Cell 3, select the `"discoverymethod"` column and count how many times each method appears by running `print(df_raw["discoverymethod"].value_counts())`.
2. **Generate summary statistics for all number columns:** On the final line of Cell 3, run `df_raw.describe().round(2)` so Jupyter renders a formatted summary table (`count`, `mean`, `std`, `min`, `25%`, `50%`, `75%`, `max`) for all 9 numeric columns.
3. **Audit the minimums, medians, means, and maximums:** Compare the `50%` (median) row against the `mean` (average) and `max` rows to identify which columns are heavily skewed by extreme physical outliers.
4. **Save and commit to Git:** Save the notebook and commit (`feat(eda): add numeric summary statistics and discovery method counts in cell 3`).

---

## PART II: Detailed Breakdown, Q&A, and Why We Made Each Decision

### Step 1.0: Project Setup, Operating System, and Git Questions

#### Q1: Why do this Exoplanet project before the Neo-Assyrian texts project?

- **The Reason:** The NASA Exoplanet dataset is already organized into a neat, flat spreadsheet (rows and columns of numbers and simple labels). This lets you focus 100% on learning core `pandas` commands. Text datasets (like damaged cuneiform letters) require heavy string cleanup and regex parsing before you even have a usable table. Starting with clean rows and columns builds your core `pandas` confidence first.

#### Q2: Should I build this on native Windows or inside an Ubuntu Virtual Machine (VM)?

- **The Reason:** On a computer with 8GB of RAM, running a full graphical Ubuntu Virtual Machine splits your memory in half (giving ~4GB to Windows and ~4GB to Ubuntu), which causes VS Code and Jupyter to lag. Because `pandas`, `numpy`, and `matplotlib` work identically on Windows and Linux, working directly in **native Windows** gives your code access to all 8GB of RAM.

#### Q3: Is PowerShell okay to use, and should I run it standalone or inside VS Code?

- **The Reason:** PowerShell works great for Python and Git. Always use the **terminal built into VS Code** rather than a separate PowerShell window because VS Code automatically starts the terminal inside your project folder and automatically activates your `(venv)` environment whenever you open a new tab.

#### Q4: Why does the virtual environment command say `venv` twice (`python -m venv venv`)?

- **The Reason:**
  - `python -m venv` tells Python: _"Run the built-in module (`-m`) named `venv`."_
  - The second `venv` is simply the **name of the folder** you want to create on your computer.
  - We use a virtual environment so that the packages we install for this project stay inside this folder and never clash with other Python projects on your computer.

#### Q5: Why did we list only 6 packages in `requirements.txt` instead of running `pip freeze`?

- **The Reason:** When you install a big library like `pandas`, `pip` automatically installs dozens of tiny helper packages behind the scenes. Putting only the 6 main libraries we actually use (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, and `ipykernel`—which lets VS Code run Jupyter Notebook cells) keeps `requirements.txt` clean, readable, and easy to install on any machine.

#### Q6: How important is the `pip` upgrade warning (`24.0 -> 26.3`)?

- **The Reason:** Low priority. When Python creates a new `venv`, it copies the default version of `pip` that originally came with your Python installer. Upgrading `pip` is optional, though running `python.exe -m pip install --upgrade pip` clears the notice and helps download newer pre-packaged libraries faster.

#### Q7: Why does `git branch -M main` happen _after_ the first commit?

- **The Reason:** When you run `git init` in an empty folder, Git prepares to track files, but no branch technically exists until the moment you save your first commit (`git commit -m "..."`). Running `git branch -M main` right after that first commit guarantees your primary branch is named `main` before you push it to GitHub.

---

### Step 1.1: Understanding the Data Download and Caching Code (Cell 1)

#### Q8: Why cannot Python combine a string and a tuple using `+` in `print()`, and how do we fix it?

- **The Reason:** Unlike some languages that automatically convert numbers or tuples into text when you use `+`, Python requires you to be explicit so you don't accidentally mix up math addition and text joining. The cleanest way to print text and variables together is using an **f-string**: put an `f` before the opening quote and put any variable inside curly braces, like `print(f"Dataset dimensions: {df_raw.shape}")`.

#### Q9: What does the Git warning `LF will be replaced by CRLF` mean?

- **The Reason:** Linux/Mac computers use one invisible character (`LF`) to mark the end of a line, while Windows uses two (`CRLF`). This warning simply means Git is automatically translating the file's invisible line endings to match Windows. It is completely harmless and does not affect your code.

#### Q10: Why does copying a table from a Jupyter Notebook produce either mashed text without spaces or raw HTML `<table>` code?

- **The Reason:** When you put `df_raw.head()` on the last line of a cell without `print()`, Jupyter renders an HTML webpage table for visual readability. Copying that visual table grabs either the unspaced HTML cells or the raw HTML source code. Whenever you want clean, space-aligned plain text to copy and paste, wrap the command in `print()`, like `print(df_raw.head())`.

#### 1. Why did we pick the `pscomppars` table from NASA instead of the `ps` table?

- NASA's standard `ps` table adds a new row every time a scientific paper is published about a planet—meaning a famous planet like `TRAPPIST-1 e` appears over a dozen times.
- The **`pscomppars` (Composite Parameters)** table combines the best measurements into **strictly 1 row per confirmed planet** (`6,470` unique planets total). Always verify what one row represents before doing math on a dataset.

#### 2. Why use `os.path.exists()` to save a local CSV copy?

- Downloading from NASA takes several seconds and fails if NASA's servers go down. Once saved as `exoplanets_raw.csv` on your computer, loading it takes less than a tenth of a second, works offline, and guarantees your total planet count (`6,470`) won't shift mid-project.

#### 3. Why do we pass `index=False` inside `.to_csv()`?

- Every `pandas` table automatically numbers its rows (`0, 1, 2, 3...`) down the left side. Passing `index=False` tells `pandas` not to save those row numbers as an extra unnamed column in the CSV file.

---

### Step 2.1: Structural Inspection & Missing Value Audit (Cell 2)

#### Q11: Why did my initial Cell 2 output look misaligned on the first line?

- **The Reason:** `df_raw.isna().sum()` outputs 12 separate lines of text. Putting `{df_raw.isna().sum()}` right after `"Total missing rows in each column: "` without a line break puts the first column (`pl_name 0`) on the same line as the title and drops the remaining 11 columns underneath. Adding `\n` (newline) before `{...}` fixes the alignment.

#### Q12: How can we prevent VS Code from truncating long outputs without changing global VS Code settings?

- **The Reason:** Printing two separate 12-line Series (`isna().sum()` and `isna().mean()`) on top of `df_raw.info()` creates nearly 50 lines of plain text, triggering VS Code's 30-line cell output limit. By combining `missing_count` and `missing_pct` into a single 2-column DataFrame (`pd.DataFrame({"Missing Count": missing_count, "Missing %": missing_pct})`) and placing it on the last line of the cell, we cut the vertical height in half, drop the redundant `dtype:` footers, and render a clean HTML table.

#### 1. What We Found in Our Missing Value Audit (Out of 6,470 Total Planets)

- **0% Missing (100% Complete):** `pl_name`, `hostname`, `discoverymethod`, `disc_year`, `sy_snum`, and `sy_pnum` have all `6,470` rows filled in.
- **Top 3 Columns with Missing Values (`NaN`):**
  1. **`pl_eqt` (Planet Equilibrium Temperature): 540 missing rows (8.35% missing).** Calculating a planet's temperature requires knowing both the host star's brightness/size and the planet's exact orbital distance. If either is unknown, `pl_eqt` cannot be calculated.
  2. **`pl_orbper` (Orbital Period): 356 missing rows (5.50% missing).** Planets discovered via Direct Imaging (orbiting far away from their star over hundreds of years) or Microlensing (a one-time gravity lens event) often do not have a measured orbital period.
  3. **`st_teff` (Star Surface Temperature): 309 missing rows (4.78% missing).** Very faint microlensing host stars or exotic remnants (like pulsars/white dwarfs) often lack standard optical temperature readings.
- **Nearly Complete Physical Columns:** `pl_rade` (only 50 missing / 0.77%), `pl_bmasse` (only 31 missing / 0.48%), and `sy_dist` (only 28 missing / 0.43%).

---

### Step 2.2: Summary Statistics & Hunting for Skew/Outliers (Cell 3)

#### 1. Why run `.value_counts()` on `discoverymethod`?

- `.describe()` only analyzes number columns (`int64` and `float64`) and ignores text columns (`str`). For categorical text columns like `discoverymethod`, `.value_counts()` is the primary tool to see how rows are distributed across categories and whether one category dominates the dataset.

#### 2. What does `df_raw.describe().round(2)` tell us?

- It calculates 8 key summary numbers for every numeric column:
  - `count`: Number of non-empty rows.
  - `mean`: The standard mathematical average (sum divided by count).
  - `std`: Standard deviation (how spread out the numbers are).
  - `min`: The smallest value in the column (used to check for impossible `0` or negative numbers).
  - `25%`, `50%` (**Median**), `75%`: The quartiles (the middle values when all planets are lined up from smallest to largest).
  - `max`: The largest value in the column (used to spot extreme outliers).

#### 3. Why compare the `mean` (average) against the `50%` (median)?

- In astronomy (just like income or housing data), distributions are often heavily **skewed** by a few giant numbers. If 9 planets take `10` days to orbit their star and 1 extreme planet takes `1,000,000` days, the **median (`50%`)** stays at `10` days, while the **mean (`average`)** gets dragged up to `100,009` days. Comparing `mean` vs. `50%` immediately reveals which columns are distorted by extreme outliers.
