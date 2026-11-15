# NASA Exoplanet Archive: Exploratory Data Science Lab (`exoplanet_eda_lab`)

An exploratory data analysis (EDA) project examining the confirmed exoplanet population (**6,470 planets**) from the NASA Exoplanet Archive's Composite Parameters (`pscomppars`) table. Built from scratch using `pandas`, `numpy`, `matplotlib`, `seaborn`, and `scipy` to practice the four core pillars of data science:

1. **Ingestion & Capture**
2. **Preparation & Cleaning**
3. **Exploration & Analysis**
4. **Communication & Action**

---

## Data Dictionary (12 Selected Columns)

| Column Code           | Plain-English Name      | Data Type | Units / Meaning                                                                                                               |
| :-------------------- | :---------------------- | :-------- | :---------------------------------------------------------------------------------------------------------------------------- |
| **`pl_name`**         | Planet Name             | `str`     | Official NASA identifier for the planet (1 unique row per planet).                                                            |
| **`hostname`**        | Host Star Name          | `str`     | Name of the star the planet orbits.                                                                                           |
| **`discoverymethod`** | Discovery Method        | `str`     | Technique used to find the planet (`Transit`, `Radial Velocity`, `Imaging`, `Microlensing`, etc.).                            |
| **`disc_year`**       | Discovery Year          | `int64`   | Year the planet discovery was published.                                                                                      |
| **`sy_snum`**         | Stars in System         | `int64`   | Total number of stars in the planetary system (`1` = single star like our Sun, `2` = binary star system, etc.).               |
| **`sy_pnum`**         | Planets in System       | `int64`   | Total number of confirmed planets orbiting this system.                                                                       |
| **`pl_orbper`**       | Orbital Period          | `float64` | Length of the planet's year in **Earth days** (time to complete 1 full orbit).                                                |
| **`pl_rade`**         | Planet Radius           | `float64` | Physical size (radius) of the planet in **Earth radii ($R_\oplus$)** (`1.0` = exact size of Earth; `11.2` $\approx$ Jupiter). |
| **`pl_bmasse`**       | Planet Mass             | `float64` | Mass (weight) of the planet in **Earth masses ($M_\oplus$)** (`1.0` = exact mass of Earth; `317.8` $\approx$ Jupiter).        |
| **`pl_eqt`**          | Planet Equilibrium Temp | `float64` | Estimated surface/equilibrium temperature of the planet in **Kelvin ($K$)** (`255 K` $\approx$ Earth's equilibrium temp).     |
| **`st_teff`**         | Star Surface Temp       | `float64` | Surface temperature of the host star in **Kelvin ($K$)** (`5778 K` = our Sun; `< 4000 K` = Red Dwarf).                        |
| **`sy_dist`**         | System Distance         | `float64` | Distance from Earth to the planetary system in **parsecs ($\text{pc}$)** (`1 parsec` $\approx 3.26$ light-years).             |
