# NASA Exoplanet Archive: Exploratory Data Science & SQLite Lab (`exoplanet_eda_lab`)

An end-to-end exploratory data analysis (EDA) and relational database pipeline examining the confirmed exoplanet population (**6,470 raw planets $\rightarrow$ 6,019 cleaned physical records**) from the NASA Exoplanet Archive's Composite Parameters (`pscomppars`) table.

Built from scratch using `pandas`, `numpy`, `matplotlib`, `seaborn`, and `sqlite3` across the four core pillars of data science:

1. **Pillar 1: Ingestion & Capture** — Programmatic TAP API CSV retrieval with offline disk caching (`exoplanets_raw.csv`).
2. **Pillar 2: Preparation & Cleaning** — Schema verification, missingness auditing, distribution skew diagnostics, and physical completeness filtering (`exoplanets_clean.csv`).
3. **Pillar 3: Exploration & Analysis** — Observational bias quantification via grouped named aggregations and physical feature engineering (`density_gcc` and `size_class`).
4. **Pillar 4: Communication & Action** — Multi-panel scientific visualization (`exoplanet_distributions.png`) and relational database loading/querying via SQLite (`exoplanets.db`).

---

## Key Analytical Findings

- **Observational Bias Across Discovery Methods:** `Transit` (74.2%) and `Radial Velocity` (18.6%) account for 92.8% of confirmed exoplanets. Grouped medians reveal that `Transit` strongly favors compact, close-in sub-Neptunes (median period `8.01` days, median mass `6.98 M_E`), whereas `Radial Velocity` favors heavier Jupiter-class gas giants on wider orbits (median period `319.70` days, median mass `314.65 M_E`).
- **Bulk Density & The `2.0 R_E` Atmospheric Threshold:** Engineered bulk density (`density_gcc`) confirms that `Earth-sized (<1.25 R_E)` and `Super-Earth (1.25–2.0 R_E)` worlds have high rocky densities (`5.67 g/cm³` and `4.43 g/cm³`), whereas planets crossing `2.0 R_E` (`Sub-Neptunes`) drop sharply in median density (`2.31 g/cm³`) as they retain volatile hydrogen/helium envelopes.
- **The Fulton Radius Valley & Gas Giant Compression:** Visual inspection of small exoplanet radii (`< 5.0 R_E`) exposes a bimodal distribution split at `~1.8–2.0 R_E` due to stellar photoevaporation. On a log-log Mass-Radius plot, gas giants (`> 100 M_E`) plateau horizontally at `~11–13 R_E` due to gravitational self-compression and electron degeneracy pressure.
- **Nearby Rocky Worlds SQL Query:** Querying the local SQLite database (`exoplanets.db`) for high-density rocky worlds (`pl_rade <= 2.0 R_E`, `density_gcc >= 4.0 g/cm³`) within `30 pc` identifies `Proxima Cen d` and Earth-twin `Proxima Cen b` (`1.30 pc`) followed by the four sub-Earth planets of `Barnard's Star` (`1.83 pc`), all detected via `Radial Velocity`.

---

## Data Dictionary (12 Curated Source Columns + 2 Engineered Columns)

| Column Code                      | Plain-English Name      | Data Type  | Units / Meaning                                                                                                 |
| :------------------------------- | :---------------------- | :--------- | :-------------------------------------------------------------------------------------------------------------- |
| **`pl_name`**                    | Planet Name             | `str`      | Official NASA identifier for the planet (1 unique row per planet).                                              |
| **`hostname`**                   | Host Star Name          | `str`      | Name of the star the planet orbits.                                                                             |
| **`discoverymethod`**            | Discovery Method        | `str`      | Technique used to find the planet (`Transit`, `Radial Velocity`, `Imaging`, `Microlensing`, etc.).              |
| **`disc_year`**                  | Discovery Year          | `int64`    | Year the planet discovery was published.                                                                        |
| **`sy_snum`**                    | Stars in System         | `int64`    | Total number of stars in the planetary system (`1` = single star like our Sun).                                 |
| **`sy_pnum`**                    | Planets in System       | `int64`    | Total number of confirmed planets orbiting this system.                                                         |
| **`pl_orbper`**                  | Orbital Period          | `float64`  | Length of the planet's year in **Earth days** (time to complete 1 full orbit).                                  |
| **`pl_rade`**                    | Planet Radius           | `float64`  | Physical size (radius) of the planet in **Earth radii ($R_\oplus$)** (`1.0` = Earth; `11.2` $\approx$ Jupiter). |
| **`pl_bmasse`**                  | Planet Mass             | `float64`  | Mass (weight) of the planet in **Earth masses ($M_\oplus$)** (`1.0` = Earth; `317.8` $\approx$ Jupiter).        |
| **`pl_eqt`**                     | Planet Equilibrium Temp | `float64`  | Estimated equilibrium temperature of the planet in **Kelvin ($K$)** (`255 K` $\approx$ Earth).                  |
| **`st_teff`**                    | Star Surface Temp       | `float64`  | Surface temperature of the host star in **Kelvin ($K$)** (`5778 K` = our Sun; `< 4000 K` = Red Dwarf).          |
| **`sy_dist`**                    | System Distance         | `float64`  | Distance from Earth to the planetary system in **parsecs ($\text{pc}$)** (`1 pc` $\approx 3.26$ light-years).   |
| **`density_gcc`** _(Engineered)_ | Bulk Planet Density     | `float64`  | Physical density in **$\text{g/cm}^3$** calculated via $(M_\oplus / R_\oplus^3) \times 5.51$.                   |
| **`size_class`** _(Engineered)_  | Planet Size Category    | `category` | Binned radius tier (`Earth-sized`, `Super-Earth`, `Sub-Neptune`, `Gas Giant`).                                  |
