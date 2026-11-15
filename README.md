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

---

## Exoplanet Discovery Methods Glossary

| Discovery Method                           | Count (Share) | How It Works Physically                                                                                                                                                                    | What It Measures Best                                                                                                 |
| :----------------------------------------- | :------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| **Transit**                                | 4,799 (74.2%) | Measures the slight dip in a star's brightness when an orbiting planet crosses directly in front of the star's disk from our line of sight.                                                | **Planet Radius (`pl_rade`)** and **Orbital Period (`pl_orbper`)**. Favors planets orbiting close to their stars.     |
| **Radial Velocity**                        | 1,205 (18.6%) | Detects the gravitational "wobble" of a star being tugged back and forth by an orbiting planet, measured via red/blue Doppler shifts in the star's spectrum.                               | **Planet Minimum Mass (`pl_bmasse`)** and **Orbital Period (`pl_orbper`)**. Favors heavy planets.                     |
| **Microlensing**                           | 294 (4.5%)    | Uses Einstein's gravitational lensing when a foreground star passes directly in front of a distant background star, magnifying the light and revealing planets around the foreground star. | Distant planets far from their stars (often thousands of parsecs from Earth), though the alignment only happens once. |
| **Imaging**                                | 98 (1.5%)     | Captures direct infrared photographs of planets by blocking out the blinding glare of the host star with a coronagraph.                                                                    | Young, hot, massive gas giants orbiting very far from their stars (`pl_orbper` of decades to ribuan of years).        |
| **Timing Variations (TTV / ETV / Pulsar)** | 56 (0.9%)     | Detects unseen planets by measuring tiny early/late arrival times in a known planet's transits, binary star eclipses, or millisecond radio pulses from a neutron star.                     | Multi-planet gravitational interactions and ultra-low-mass objects.                                                   |
| **Orbital Brightness / Astrometry / Disk** | 18 (0.3%)     | Detects planets via starlight reflection phases, tiny side-to-side sky position shifts of the star (`Astrometry`), or gaps carved in protoplanetary dust disks.                            | Massive planets or newly forming worlds.                                                                              |

---

## Summary Statistics Terms (`.describe()`)

- **`count`**: Total number of non-empty (`non-NaN`) measurements in the column.
- **`mean`**: Mathematical average (sum divided by `count`). Strongly pulled upward or downward by extreme outliers.
- **`std` (Standard Deviation)**: Measures the typical spread or dispersion of numbers around the `mean`. A small `std` indicates values are tightly clustered; a large `std` indicates wide variability or extreme outliers.
- **`min` / `max`**: The smallest and largest recorded values in the column.
- **`25%` / `50%` (Median) / `75%`**: Quartile cutoffs when values are sorted from smallest to largest. The **`50%` median** represents the true middle planet and is resistant to extreme outliers.
